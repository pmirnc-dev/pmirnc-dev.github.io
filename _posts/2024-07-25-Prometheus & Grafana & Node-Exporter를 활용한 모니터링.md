---
layout: post
title: "Prometheus & Grafana & Node-Exporter를 활용한 모니터링"
date: 2024-07-25 15:00:00 +0900
author: jseom
categories: Prometheus Grafana Node-Exporter Docker-compose Linux
published: true
---

<hr />

실시간 서버 모니터링을 위해 찾아본 오픈소스 도구들입니다. 현업에서도 사용중이기 때문에, 신뢰성이 높습니다.

1. **Prometheus**: 시계열 데이터베이스로, 서버 및 애플리케이션의 시계열 데이터(매트릭)을 수집하고 저장합니다.
2. **Grafana**: 시각화 도구로, Prometheus에서 수집한 데이터를 기반으로 대시보드를 만들어 실시간 모니터링을 가능하게 합니다.
3. **Node-Exporter**: 서버의 시계열 데이터를 수집하여 Prometheus에 제공하는 에이전트입니다.

---

### 설치 및 설정

#### **Node-Exporter 설치**

  <!--Node-Exporter 다운로드 및 설치-->

sudo adduser prometheus

wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz

tar -xvf node_exporter-1.8.2.linux-amd64.tar.gz

mv node_exporter-1.8.2.linux-amd64 node_exporter

exit

  <!--Node-Exporter 시스템 서비스 등록-->

sudo su

vi /etc/systemd/system/node_exporter.service

---

[Unit]
Description=Prometheus Node Exporter
Documentation=https://prometheus.io/docs/guides/node-exporter/
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Restart=on-failure
ExecStart=/home/prometheus/node_exporter/node_exporter

[Install]
WantedBy=multi-user.target

---

  <!--node_exporter 서비스 확인-->

systemctl daemon-reload

systemctl start node_exporter

systemctl status node_exporter

systemctl enable node_exporter

  <!--실행 여부 체크-->

curl -X GET http://localhost:9100/metrics

#### **Prometheus & Grafana 설치(docker compose 활용)**

  <!--Prometheus 설치 전 작업-->

mkdir grafana

sudo chmod 777 grafana

mkdir -p prometheus/conifg

vi prometheus/config/prometheus.yml

---

global:
scrape_interval: 15s
evaluation_interval: 15s

scrape_configs:

- job_name: 'node_exporter'
  static_configs:
  - targets: ['setip:9100']

---

sudo chmod -R 777 prometheus/

  <!--Prometheus 다운로드 및 설치-->

vi docker-compose.yml

---

version: '3.8'

services:
prometheus:
image: prom/prometheus
container_name: prometheus
ports: - "9090:9090"
volumes: - /path:/prometheus - /path:/etc/prometheus/prometheus.yml
command: - "--config.file=/etc/prometheus/prometheus.yml" - "--storage.tsdb.retention.time=1y" - "--web.enable-lifecycle" - "--web.enable-admin-api"
restart: always

grafana:
image: "grafana/grafana"
container_name: grafana
ports: - "3000:3000"
volumes: - /path:/var/lib/grafana
restart: always

---

sudo docker-compose up

---

### 대시보드 설정

1. Grafana에 접속하여 로그인합니다.
   1. 초기 ID 및 비밀번호: admin/admin
2. 데이터 소스로 Prometheus를 추가합니다.
   1. Home → Connection → Data sources
   2. Connection URL에 Prometheus의 URL 입력
   3. 최하단 Save & test로 확인
3. Node-Exporter를 통해 수집된 메트릭을 기반으로 대시보드를 생성합니다.
   1. Home → Dashboards → Create dashboard
      1. Node-Exporter의 경우, [Node Exporter Full](https://grafana.com/grafana/dashboards/1860-node-exporter-full/) 대시보드를 추천
      2. json 파일을 업로드 하거나, ID명 “1860” 입력, Load
      3. 최하단 Prometheus 링크 선택

---

## 참고

[https://sanggi-jayg.tistory.com/entry/Prometheus-Node-Exporter-Grafana-연동](https://sanggi-jayg.tistory.com/entry/Prometheus-Node-Exporter-Grafana-%EC%97%B0%EB%8F%99)
