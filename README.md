# Ansible Role: Nginx

Docker Compose를 사용하여 Nginx 리버스 프록시를 배포하고 설정하는 Ansible Role입니다. 다양한 DevOps 도구(Jenkins, GitLab, ArgoCD, Grafana, Prometheus 등)에 대한 리버스 프록시 설정을 지원합니다.

## 📋 목차

- [개요](#개요)
- [주요 기능](#주요-기능)
- [요구사항](#요구사항)
- [설치 방법](#설치-방법)
- [사용 방법](#사용-방법)
- [변수 설정](#변수-설정)
- [지원하는 서비스](#지원하는-서비스)
- [디렉토리 구조](#디렉토리-구조)
- [예제](#예제)
- [라이선스](#라이선스)

## 개요

이 Ansible Role은 Nginx를 Docker 컨테이너로 배포하고, 다양한 DevOps 도구에 대한 리버스 프록시를 자동으로 설정합니다. Jinja2 템플릿을 사용하여 유연하게 설정을 관리할 수 있으며, 각 서비스별로 독립적으로 활성화/비활성화가 가능합니다.

## 주요 기능

- **Docker Compose 기반 배포**: Nginx를 컨테이너로 간편하게 배포 및 관리
- **리버스 프록시 설정**: 다양한 DevOps 도구에 대한 리버스 프록시 자동 설정
- **유연한 설정**: 서비스별 독립적인 활성화/비활성화 가능
- **헬스체크 엔드포인트**: `/healthcheck`, `/nginx_status` 엔드포인트 제공
- **보안 헤더**: X-Frame-Options, X-XSS-Protection, X-Content-Type-Options 자동 설정
- **로그 관리**: 액세스 로그 및 에러 로그 볼륨 마운트
- **SSL/TLS 지원**: HTTPS 포트 설정 및 인증서 관리
- **Upstream 지원**: 로드 밸런싱을 위한 upstream 서버 설정
- **네트워크 설정**: Docker 네트워크 및 Extra Hosts 설정 지원

## 요구사항

### 플랫폼

이 Role은 다음 플랫폼을 지원합니다:

- **Red Hat Enterprise Linux** (모든 버전)
- **Debian** (모든 버전)
- **Ubuntu** (모든 버전)
- **Alpine Linux** (모든 버전)
- **Arch Linux** (모든 버전)

### Ansible 버전

- Ansible 2.1 이상

### 의존성

- **codesuiteapp.user**: 사용자 및 그룹 생성을 위한 Role
  - Repository: https://github.com/codesuiteapp/ansible-role-user.git
  - Version: main

### 필수 소프트웨어

- Docker
- Docker Compose

## 설치 방법

### Ansible Galaxy를 통한 설치 (예정)

```bash
ansible-galaxy install codesuiteapp.nginx
```

### Git을 통한 설치

```bash
git clone https://github.com/codesuiteapp/ansible-role-nginx.git
```

### requirements.yml에 추가

```yaml
- name: codesuiteapp.nginx
  src: https://github.com/codesuiteapp/ansible-role-nginx.git
  version: main
```

설치:
```bash
ansible-galaxy install -r requirements.yml
```

## 사용 방법

### 기본 사용

```yaml
- hosts: servers
  roles:
    - role: codesuiteapp.nginx
      vars:
        nginx_http_port: 8080
        nginx_https_port: 8443
```

### 특정 서비스 활성화

```yaml
- hosts: servers
  roles:
    - role: codesuiteapp.nginx
      vars:
        # Jenkins 프록시 활성화
        use_jenkins: true
        jenkins_port: 9753
        jenkins_url: "http://localhost:9753/jenkins"

        # GitLab 프록시 활성화
        use_gitlab: true
        gitlab_url: "http://localhost:9999/gitlab"

        # Grafana 프록시 활성화
        use_grafana: true
        grafana_url: "http://localhost:3000/grafana"
        grafana_domain: "grafana.example.com"
```

## 변수 설정

### 기본 설정

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `image_registry_server` | `docker.io` | Docker 이미지 레지스트리 서버 |
| `nginx_image` | `library/nginx` | Nginx 이미지 이름 |
| `nginx_ver` | `latest` | Nginx 이미지 버전 |
| `nginx_port` | `8080` | Nginx 서비스 포트 |
| `docker_compose_bin` | `docker-compose` | Docker Compose 바이너리 경로 |

### 디렉토리 설정

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `app_home_dir` | `/app` | 애플리케이션 홈 디렉토리 |
| `app_data_dir` | `/data` | 애플리케이션 데이터 디렉토리 |
| `app_logs_dir` | `/log` | 애플리케이션 로그 디렉토리 |
| `nginx_home` | `{{ app_home_dir }}/nginx-docker` | Nginx 홈 디렉토리 |
| `nginx_conf` | `{{ nginx_home }}/conf` | Nginx 설정 디렉토리 |
| `nginx_cert` | `{{ nginx_home }}/cert` | Nginx 인증서 디렉토리 |
| `nginx_logs` | `{{ app_logs_dir }}/nginx` | Nginx 로그 디렉토리 |

### 포트 설정

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `nginx_http_port` | `8080` | HTTP 포트 |
| `nginx_https_port` | `8443` | HTTPS 포트 |

### 네트워크 설정

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `use_dkr_net` | `false` | Docker 네트워크 사용 여부 |
| `dkr_network` | `devops_net` | Docker 네트워크 이름 |
| `use_extra_hosts` | `false` | Extra Hosts 사용 여부 |
| `add_etc_host` | `10.11.22.10 devops.com` | 추가할 호스트 |

### Upstream 설정

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `use_upstream` | `false` | Upstream 사용 여부 |
| `upstream_name` | `devops` | Upstream 이름 |
| `upstream_servers` | `[127.0.0.1]` | Upstream 서버 목록 |

### SSL 설정

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `redirect_ssl` | `false` | SSL 리다이렉트 사용 여부 |
| `use_traefik` | `false` | Traefik 사용 여부 |
| `use_traefik_tls` | `false` | Traefik TLS 사용 여부 |

### 사용자 설정

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `create_user` | `false` | 사용자 생성 여부 |
| `adm_user` | `appadm` | 관리자 사용자명 |
| `adm_group` | `appadm` | 관리자 그룹명 |
| `adm_uid` | `1000` | 관리자 UID |

## 지원하는 서비스

### Jenkins

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `use_jenkins` | `false` | Jenkins 프록시 활성화 |
| `jenkins_port` | `9753` | Jenkins 포트 |
| `jenkins_url` | `http://localhost:9753/jenkins` | Jenkins URL |
| `redirect_jenkins_port` | `8443` | Jenkins 리다이렉트 포트 |
| `redirect_jenkins_url` | `http://cicd.sto-devops.koscom.co.kr:8080` | Jenkins 리다이렉트 URL |

### GitLab

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `use_gitlab` | `false` | GitLab 프록시 활성화 |
| `gitlab_url` | `http://localhost:9999/gitlab` | GitLab URL |
| `redirect_gitlab_url` | `http://cicd.sto-devops.koscom.co.kr` | GitLab 리다이렉트 URL |

### ArgoCD

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `use_argocd` | `false` | ArgoCD 프록시 활성화 |
| `argocd_url` | `http://localhost:80/argocd` | ArgoCD URL |
| `argocd_domain` | `argocd.sto-dev.koscom.co.kr` | ArgoCD 도메인 |

### Grafana

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `use_grafana` | `false` | Grafana 프록시 활성화 |
| `grafana_url` | `http://localhost:3000/grafana` | Grafana URL |
| `grafana_domain` | `grafana.sto-dev.koscom.co.kr` | Grafana 도메인 |

### Prometheus

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `use_prometheus` | `false` | Prometheus 프록시 활성화 |
| `prometheus_url` | `http://localhost:9090/prometheus` | Prometheus URL |
| `prometheus_domain` | `prometheus.sto-dev.koscom.co.kr` | Prometheus 도메인 |

### Nexus

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `use_nexus` | `false` | Nexus 프록시 활성화 |
| `nexus_url` | `http://localhost:8081/nexus` | Nexus URL |

### AWX

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `use_awx` | `false` | AWX 프록시 활성화 |
| `awx_url` | `http://localhost:8888/` | AWX URL |
| `awx_domain_name` | `awx.sto-devops.koscom.co.kr` | AWX 도메인 |

### Portainer

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `use_portainer` | `false` | Portainer 프록시 활성화 |
| `portainer_url` | `http://localhost:9000/portainer` | Portainer URL |
| `portainer_domain` | `portainer.sto-devops.koscom.co.kr` | Portainer 도메인 |

### Flame

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `use_flame` | `true` | Flame 프록시 활성화 |
| `flame_url` | `http://localhost:5005/` | Flame URL |
| `flame_domain_name` | `home.sto-devops.koscom.co.kr` | Flame 도메인 |

### Batch

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `use_batch` | `false` | Batch 프록시 활성화 |
| `batch_port` | `9876` | Batch 포트 |
| `batch_url` | `http://localhost:9876/batch` | Batch URL |
| `redirect_batch_url` | `http://cicd.sto-devops.koscom.co.kr:8080` | Batch 리다이렉트 URL |

### Jira

| 변수명 | 기본값 | 설명 |
|--------|--------|------|
| `use_jira` | `false` | Jira 프록시 활성화 |
| `jira_url` | `http://localhost:8080/jira` | Jira URL |

## 디렉토리 구조

```
ansible-role-nginx/
├── defaults/           # 기본 변수 정의
│   └── main.yml
├── files/             # 정적 파일
│   ├── cert/          # SSL/TLS 인증서
│   └── conf/          # Nginx 설정 파일
├── handlers/          # Ansible 핸들러
│   └── main.yml
├── meta/              # Role 메타데이터
│   └── main.yml
├── tasks/             # Ansible 태스크
│   └── main.yml
├── templates/         # Jinja2 템플릿
│   ├── .env.j2                    # Docker Compose 환경 변수
│   ├── docker-compose.yml.j2      # Docker Compose 설정
│   └── default.conf.j2            # Nginx 서버 블록 설정
├── tests/             # 테스트 파일
│   └── test.yml
├── vars/              # 변수 정의
│   └── main.yml
├── LICENSE            # 라이선스 파일
└── README.md          # 본 문서
```

## 예제

### 예제 1: 기본 Nginx 배포

```yaml
---
- hosts: web_servers
  become: yes
  roles:
    - role: codesuiteapp.nginx
      vars:
        nginx_http_port: 80
        nginx_https_port: 443
        env: production
```

### 예제 2: Jenkins와 GitLab 리버스 프록시

```yaml
---
- hosts: devops_servers
  become: yes
  roles:
    - role: codesuiteapp.nginx
      vars:
        # Jenkins 설정
        use_jenkins: true
        jenkins_port: 8080
        jenkins_url: "http://localhost:8080/jenkins"

        # GitLab 설정
        use_gitlab: true
        gitlab_url: "http://localhost:9999/gitlab"
        redirect_gitlab_url: "http://gitlab.example.com"

        # 네트워크 설정
        use_dkr_net: true
        dkr_network: devops_network
```

### 예제 3: 모니터링 스택 (Grafana + Prometheus)

```yaml
---
- hosts: monitoring_servers
  become: yes
  roles:
    - role: codesuiteapp.nginx
      vars:
        # Grafana 설정
        use_grafana: true
        grafana_url: "http://localhost:3000/grafana"
        grafana_domain: "grafana.monitoring.example.com"

        # Prometheus 설정
        use_prometheus: true
        prometheus_url: "http://localhost:9090/prometheus"
        prometheus_domain: "prometheus.monitoring.example.com"

        # SSL 설정
        redirect_ssl: true
        nginx_https_port: 443
```

### 예제 4: Upstream을 사용한 로드 밸런싱

```yaml
---
- hosts: load_balancers
  become: yes
  roles:
    - role: codesuiteapp.nginx
      vars:
        use_upstream: true
        upstream_name: web_backend
        upstream_servers:
          - 192.168.1.10:8080
          - 192.168.1.11:8080
          - 192.168.1.12:8080
```

## 헬스체크 엔드포인트

이 Role은 다음과 같은 헬스체크 엔드포인트를 제공합니다:

- **`/healthcheck`**: JSON 형식의 헬스체크 응답 (`{"status": "Healthy"}`)
- **`/nginx_status`**: Nginx stub_status 모듈을 통한 상태 정보
- **`/server_status`**: 서버 상태 JSON 응답

## 보안 고려사항

### 기본 보안 헤더

이 Role은 다음과 같은 보안 헤더를 자동으로 설정합니다:

- `X-Frame-Options: SAMEORIGIN` - 클릭재킹 공격 방지
- `X-XSS-Protection: 1; mode=block` - XSS 공격 방지
- `X-Content-Type-Options: nosniff` - MIME 타입 스니핑 방지

### 프록시 헤더

리버스 프록시 동작을 위해 다음 헤더를 설정합니다:

- `X-Real-IP` - 클라이언트의 실제 IP 주소
- `X-Forwarded-For` - 프록시 체인의 IP 주소들
- `X-Forwarded-Proto` - 원본 프로토콜 (http/https)
- `X-Forwarded-Host` - 원본 호스트
- `X-Forwarded-Port` - 원본 포트

## 문제 해결

### Docker Compose가 실행되지 않는 경우

```bash
# Docker Compose 버전 확인
docker-compose --version

# Docker 서비스 상태 확인
systemctl status docker

# Nginx 컨테이너 로그 확인
docker logs nginx-dev
```

### 권한 문제

```bash
# 디렉토리 권한 확인
ls -la /app/nginx-docker

# 사용자 및 그룹 확인
id appadm
```

### 포트 충돌

```bash
# 포트 사용 확인
netstat -tulpn | grep 8080
lsof -i :8080
```

## 기여하기

버그 리포트, 기능 제안, Pull Request는 언제나 환영합니다!

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 라이선스

이 프로젝트는 GPL-2.0-or-later 라이선스 하에 배포됩니다. 자세한 내용은 `LICENSE` 파일을 참조하세요.

## 작성자

- **CodeSuiteApp** - [https://github.com/codesuiteapp](https://github.com/codesuiteapp)

## 감사의 말

이 Role을 개발하는 데 도움을 주신 모든 기여자분들께 감사드립니다.
