---
layout: post
title:  "github action을 활용한 CI/CD(with AWS EKS)"
date:   2023-12-27 09:00:00 +0900
author: hsyoo
categories: AWS EKS k8s CI/CD github
---
<hr>

## 머리말

제가 쓰는 올해 마지막 포스팅이 되겠네요.

AWS EKS를 활용해 운영한지 벌써 1년이 넘었습니다. (EKS는 AWS에서 제공하는 k8s 서비스입니다.)

k8s에 대한 지식이 아직도 부족하지만, 그 동안 쌓은 경험들로 얼마 전 Azure AKS를 활용한 k8s 운영도 시작했습니다.

벌써 클라우드 플랫폼만 세번째네요..(AWS, NCP, Azure) 모두 k8s를 활용했습니다.

최근 github를 업무에 적극 활용하기 시작했습니다. 회사에서 MS Teams를 사용하게 되면서 그 효과가 더욱 커졌습니다.

**오늘은 그동안 미루어왔던 github action을 활용해 AWS EKS 클러스터 CI/CD 파이프라인을 구축 사례를 공유하려고 합니다.**

## 본문

### 자격 증명 공급자(Identity providers) 생성

AWS IAM에서 github action을 위한 자격 증명 공급자를 생성합니다. (아래 그림 참조)


![img.png](/assets/images/hsyoo/aws-identity-providers.png)

### IAM 정책 생성

2가지를 생성할텐데, 

private ECR(container registry)를 위한 정책 하나,

EKS 클러스터에 배포를 위한 정책 하나입니다.

![img.png](/assets/images/hsyoo/policy-json.png)

### ECR
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:GetDownloadUrlForLayer",
                "ecr:BatchGetImage",
                "ecr:CompleteLayerUpload",
                "ecr:DescribeImages",
                "ecr:GetAuthorizationToken",
                "ecr:DescribeRepositories",
                "ecr:UploadLayerPart",
                "ecr:ListImages",
                "ecr:InitiateLayerUpload",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetRepositoryPolicy",
                "ecr:PutImage"
            ],
            "Resource": "*"
        }
    ]
}
```

#### EKS
리소스에는 클러스터 ARN을 적어주세요
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "eks:*"
      ],
      "Resource": [
        "arn:aws:eks:ap-northeast-2:123456789012:cluster/클러스터이름"
      ],
      "Condition": {}
    }
  ]
}
```


### IAM 역할 생성

다음으로 역할을 생성합니다. (아래 그림 참조)

웹 자격 증명을 선택하고, 위에서 생성만 자격 증명 공급자를 선택합니다.

github 조직 이름을 적어주시고, repo와 branch는 optional입니다.

![img.png](/assets/images/hsyoo/iam-role.png)

앞에서 생성한 2가지 정책에 대한 권한을 추가해줍시다.

![img.png](/assets/images/hsyoo/policy-add.png)


### github secrets 설정

github action marketplace에 등록된 [kodermax/kubectl-aws-eks@main](https://github.com/marketplace/actions/kubectl-aws-eks){:target=_blank}을 사용할건데,

이액션을 위해 github action에서 사용될 kubeconfig 파일을 secrets을 생성해야 합니다. (윈도우 환경이기 때문에 PowerShell을 이용했습니다.)

![img.png](/assets/images/hsyoo/bas64-secrets.png)

위에서 생성한 base64문자열을 secrets에 추가해줍니다. (여기서 추가로 필요하신 건 등록해도 됩니다.)

![img.png](/assets/images/hsyoo/github-secrets.png)


### github action workflow 설정

이제 github action workflow를 설정해줍시다.

repo root 위치에서 .github > workflows 폴더를 생성하고, 그 안에 yaml 파일을 생성합니다.

과정들은 아래 yaml파일에 주석으로 달아놨습니다.

```yaml
name: Deploy to AWS EKS
on:
  push:
    branches:
      - 'deploy/**' # DEPLOY 브랜치에 푸시되었을 때 실행하겠습니다. 예시)deploy/221223
env:
  ECR_URI: 123456789012.dkr.ecr.ap-northeast-2.amazonaws.com # ECR URI로 고쳐주십시오
  ECR_REPOSITORY: pmi-api # ECR 레포지토리 이름
  ROLE_ARN: arn:aws:iam::123456789012:role/github-actions-eks # 위에서 생성한 역할의 ARN
  AWS_REGION: ap-northeast-2
  DEPLOYMENT_NAME: pmi-api  # k8s deployment 이름을 변수화 했습니다
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    permissions:
      id-token: write   # This is required for requesting the JWT
      contents: read    # This is required for actions/checkout

    steps:
      # 체크아웃 액션을 해야 github action에서 코드를 가져올 수 있습니다.(필수)
      - name: Checkout
        uses: actions/checkout@v3 

      # AWS 자격 증명 (생성한 OIDC 공급자를 사용해 역할을 얻습니다.)
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${\{ env.ROLE_ARN }}
          aws-region: ${\{ env.AWS_REGION }}

      # AWS ECR 로그인 (ECR에 이미지를 푸시하기 위해 필요합니다.)
      - name: Login to Amazon ECR Private
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2


      # docker 이미지를 빌드 하고 push 하겠습니다. image tag는 github sha를 사용합시다.
      - name: Build and push docker image to Amazon ECR
        env:
          REGISTRY: ${\{ steps.login-ecr.outputs.registry }}
          REPOSITORY: ${\{ env.ECR_REPOSITORY }}
          IMAGE_TAG: ${\{ github.sha }}
        run: |
          docker build -t $REGISTRY/$REPOSITORY:$IMAGE_TAG .
          docker push $REGISTRY/$REPOSITORY:$IMAGE_TAG

      # kubectl-aws-eks 액션을 사용해 EKS 클러스터에 있는 deployment의 image를 업데이트 합니다.
      - name: deploy to cluster
        uses: kodermax/kubectl-aws-eks@main
        env:
          KUBE_CONFIG_DATA: ${\{ secrets.AWS_PMI_EKS_KUBECONFIG }} # 위에서 생성한 kubeconfig secrets
          ECR_REGISTRY: ${\{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: ${\{ env.ECR_REPOSITORY }}
          DEPLOYMENT_NAME: ${\{ env.DEPLOYMENT_NAME }}
          IMAGE_TAG: ${\{ github.sha }}
        with:
          args: set image deployment/$DEPLOYMENT_NAME $ECR_REPOSITORY=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG

      # 이미지가 업데이트 되었으니 rollout을 해줍니다.
      - name: verify deployment
        uses: kodermax/kubectl-aws-eks@main
        env:
          KUBE_CONFIG_DATA: ${\{ secrets.AWS_PMI_EKS_KUBECONFIG }}
          DEPLOYMENT_NAME: ${\{ env.DEPLOYMENT_NAME }}
        with:
          args: rollout status deployment/$DEPLOYMENT_NAME
```

### cluster에 역할 추가

EKS 클러스터에 역할을 추가해줍니다.

clusterrole이 아닌 role로 할 수도 있지만 편의를 위해 클러스터에 역할을 주고 바인딩 하겠습니다.

clusterrole과 clusterrolebinding을 생성하는 yaml입니다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # 클러스터 role은 namepsace가 필요 없습니다
  name: github-oidc-cluster-role
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: github-oidc-cluster-role-binding
subjects:
  - kind: User
    name: github-oidc-auth-user
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: github-oidc-cluster-role
  apiGroup: rbac.authorization.k8s.io
```

여기가 끝이 아닙니다. 생성만 한다고 되는 건 아니고, aws-auth configmap에 역할을 추가해줘야 합니다.

![img.png](/assets/images/hsyoo/aws-auth.png)

```bash

kubectl edit configmap aws-auth -n kube-system

```

아래와 같이 추가해줍시다.

```yaml
apiVersion: v1
kind: ConfigMap
data:
  mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::123456789012:role/github-actions-eks # 위에서 생성한 역할의 ARN
      username: github-oidc-auth-user # 위에서 생성한 cluster role의 이름
```

### 결과

드디어 끝났습니다. 배포버전의 branch를 deploy/** 형태로 생성하고 푸시하면 github action이 실행됩니다.

![img.png](/assets/images/hsyoo/github-actions-workflow.png)

## 마치면서..

이번 포스팅은 github action을 활용해 CI/CD 파이프라인을 구축하는 방법을 공유했습니다.

꼬박 하루를 투자해야 했지만, github action을 활용해 CI/CD 파이프라인을 구축할 수 있게 되었습니다.

branch 관리를 철저히 하지 않으면, 배포 버전이 잘못 배포되는 등의 문제가 발생할 수 있습니다.

이제 앞으로 진행하는 프로젝트들은 github action을 활용해 CI/CD 파이프라인을 통해 배포를 진행할 예정입니다.

긴 글 읽어주셔서 감사하고, 이 포스팅에 큰 도움이 된 참고 자료를 남깁니다.

그럼 새해에도 모두 행복하시고 건강하시길

참고 자료

- [https://velog.io/@jeongmin78/CICD-Github-Action-AWS-IAM-Role-%EC%9D%B4%EC%9A%A9%ED%95%B4-%EC%9D%B4%EB%AF%B8%EC%A7%80%EB%A5%BC-ECR%EC%97%90-%EC%98%AC%EB%A6%AC%EA%B8%B0-8n3fmmgn](https://velog.io/@jeongmin78/CICD-Github-Action-AWS-IAM-Role-%EC%9D%B4%EC%9A%A9%ED%95%B4-%EC%9D%B4%EB%AF%B8%EC%A7%80%EB%A5%BC-ECR%EC%97%90-%EC%98%AC%EB%A6%AC%EA%B8%B0-8n3fmmgn){:target=_blank}
- [https://medium.com/uplusdevu/%EA%B9%83%ED%97%88%EB%B8%8C-%EC%95%A1%EC%85%98-github-action-%EF%B8%8E-aws-%EC%9D%B8%EC%A6%9D-openid-connect-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0-8cbe0ff4434c](https://medium.com/uplusdevu/%EA%B9%83%ED%97%88%EB%B8%8C-%EC%95%A1%EC%85%98-github-action-%EF%B8%8E-aws-%EC%9D%B8%EC%A6%9D-openid-connect-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0-8cbe0ff4434c){:target=_blank}
- [https://bhaskar422.hashnode.dev/complete-cicd-on-eks-cluster-using-github-actions-and-authenticate-using-rbac-and-oidc-provider#heading-2-assume-the-role-for-authentication-in-github](https://bhaskar422.hashnode.dev/complete-cicd-on-eks-cluster-using-github-actions-and-authenticate-using-rbac-and-oidc-provider#heading-2-assume-the-role-for-authentication-in-github){:target=_blank}


