# AGENTS.md - Ansible Role: Nginx

## 1. 🎯 Project Overview

**Type**: Ansible Role
**Purpose**: Deploy and configure Nginx as a reverse proxy using Docker Compose
**Main Use Case**: Reverse proxy for DevOps tools (Jenkins, GitLab, ArgoCD, Grafana, Prometheus, AWX, Portainer, etc.)

**Key Characteristics**:
- Infrastructure as Code (IaC) using Ansible
- Container-based deployment with Docker Compose
- Template-driven configuration using Jinja2
- Modular service activation (enable/disable individual proxies)

## 2. 🛠️ Tech Stack

- **Automation**: Ansible (min version 2.1)
- **Container Runtime**: Docker
- **Container Orchestration**: Docker Compose
- **Web Server**: Nginx (Docker image)
- **Templating**: Jinja2
- **Configuration Management**: YAML
- **Supported Platforms**: RHEL, Debian, Ubuntu, Alpine, Arch Linux

## 3. 🏗️ Architecture and Project Structure

### Directory Layout
```
ansible-role-nginx/
├── defaults/main.yml          # Default variables (88 vars)
├── files/                     # Static files
│   ├── cert/                  # SSL/TLS certificates
│   └── conf/                  # Base nginx config
├── handlers/main.yml          # Service handlers (start/restart)
├── meta/main.yml              # Role metadata & dependencies
├── tasks/main.yml             # Main task execution flow
├── templates/                 # Jinja2 templates
│   ├── .env.j2                # Docker image configuration
│   ├── docker-compose.yml.j2  # Container definition
│   └── default.conf.j2        # Nginx server blocks (297 lines)
├── tests/test.yml             # Test playbook
└── vars/main.yml              # Additional variables
```

### Execution Flow
1. **Create directories**: nginx_home, nginx_conf, nginx_logs, nginx_cert
2. **Copy static files**: cert/, conf/ directories
3. **Template generation**: .env, docker-compose.yml, default.conf
4. **Service management**: Restart nginx via systemd (triggers docker-compose)

### Key Files

#### `tasks/main.yml`
- Creates required directories with proper permissions
- Copies configuration files and certificates
- Renders Jinja2 templates
- Triggers nginx restart via systemd

#### `templates/docker-compose.yml.j2`
- Defines nginx container configuration
- Mounts volumes for configs, logs, certs
- Configures ports (nginx_http_port:8080, nginx_https_port:8443)
- Supports Docker networks and extra_hosts

#### `templates/default.conf.j2`
- Main nginx configuration template (297 lines)
- Conditional server blocks based on `use_*` flags
- Implements reverse proxy for each service
- Provides health check endpoints

#### `handlers/main.yml`
- `Start nginx`: `docker-compose up --detach`
- `Restart nginx`: `docker-compose down && up --detach`

## 4. 🔑 Core Domain Concepts

### Service Activation Pattern
Each service follows this pattern:
```yaml
use_<service>: false              # Enable/disable flag
<service>_url: "http://..."       # Backend service URL
<service>_domain: "example.com"   # Optional domain name
```

### Supported Services
- **Jenkins**: CI/CD automation server
- **GitLab**: Git repository management
- **ArgoCD**: GitOps continuous delivery
- **Grafana**: Metrics visualization
- **Prometheus**: Metrics collection
- **Nexus**: Artifact repository
- **AWX**: Ansible automation platform
- **Portainer**: Docker management UI
- **Flame**: Dashboard/homepage
- **Batch**: Batch processing service
- **Jira**: Issue tracking

### Configuration Layers
1. **Base Config** (`defaults/main.yml`): 88 variables with defaults
2. **Template Variables**: Injected into Jinja2 templates
3. **Conditional Blocks**: Services enabled via `use_*` flags
4. **Runtime Config**: Docker Compose environment variables

### Key Variables Categories

**Docker Configuration**:
- `image_registry_server`, `nginx_image`, `nginx_ver`
- `docker_compose_bin`, `use_dkr_net`

**Directory Structure**:
- `app_home_dir`, `app_data_dir`, `app_logs_dir`
- `nginx_home`, `nginx_conf`, `nginx_cert`, `nginx_logs`

**Network Configuration**:
- `nginx_http_port`, `nginx_https_port`
- `use_upstream`, `upstream_servers`
- `use_extra_hosts`, `add_etc_host`

**Security**:
- `redirect_ssl`, `use_traefik`, `use_traefik_tls`
- Security headers: X-Frame-Options, X-XSS-Protection, X-Content-Type-Options

## 5. 📜 Coding Guidelines and Rules

### Must-Do

1. **Variable Naming**: Use snake_case for all variables
2. **Service Pattern**: Always follow `use_<service>` + `<service>_url` pattern
3. **Jinja2 Templates**: Use proper whitespace control (`{%- -%}` when needed)
4. **Conditionals**: Wrap service blocks in `{% if use_<service> %}`
5. **File Permissions**: Set owner, group, mode for all files/directories
6. **Idempotency**: All tasks must be idempotent
7. **YAML Formatting**: Use consistent 2-space indentation
8. **Template Syntax**: Follow nginx configuration best practices
9. **Security Headers**: Always include security headers in proxy blocks
10. **Health Checks**: Maintain `/healthcheck` and `/nginx_status` endpoints

### Don'ts

1. **Don't hardcode values**: Use variables from defaults/main.yml
2. **Don't skip backups**: Use `backup: true` when templating
3. **Don't ignore errors**: Only use `ignore_errors: true` when explicitly needed
4. **Don't create circular dependencies**: Check meta/main.yml dependencies
5. **Don't expose sensitive data**: Use separate variables for secrets
6. **Don't use absolute paths**: Use variable-based paths (e.g., `{{ nginx_home }}`)
7. **Don't bypass systemd**: Use handlers for service management
8. **Don't mix configurations**: Keep service-specific configs in separate blocks
9. **Don't skip validation**: Test YAML syntax before committing
10. **Don't duplicate code**: Use loops and includes for repetitive tasks

### Ansible Best Practices

- Use `ansible.builtin.*` module names (not legacy short names)
- Leverage `loop` instead of `with_items`
- Set `changed_when` for shell/command tasks
- Use `register` to capture task output
- Name all tasks descriptively
- Use `become: true` only when necessary
- Validate templates with yamllint

### Nginx Configuration Patterns

**Proxy Headers (Standard)**:
```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Forwarded-Port $server_port;
```

**Health Check Endpoints**:
```nginx
location /healthcheck {
    access_log off;
    add_header 'Content-Type' 'application/json';
    return 200 '{"status": "Healthy"}';
}
```

## 6. ⚙️ Configuration and Environment

### Required Environment Variables

Set via `templates/.env.j2`:
```bash
DOCKER_IMAGE={{ image_registry_server }}/{{ nginx_image }}:{{ nginx_ver }}
trf_domain={{ traefik_domain }}
```

### Volume Mounts

```yaml
volumes:
  - {{ nginx_home }}/cert:/app/nginx/cert
  - {{ nginx_home }}/html:/etc/nginx/html
  - {{ nginx_home }}/conf/nginx.conf:/etc/nginx/nginx.conf
  - {{ nginx_home }}/conf/sites-enabled:/etc/nginx/conf.d
  - {{ nginx_logs }}:/var/log/nginx
```

### Port Mappings

- HTTP: `{{ nginx_http_port }}:8080` (default: 8080:8080)
- HTTPS: `{{ nginx_https_port }}:8443` (default: 8443:8443)

### Dependencies

**Ansible Galaxy Role**:
```yaml
dependencies:
  - name: codesuiteapp.user
    src: https://github.com/codesuiteapp/ansible-role-user.git
    version: main
```

### User Management

```yaml
create_user: false    # Set true to create user
adm_user: "appadm"    # Service user
adm_group: "appadm"   # Service group
adm_uid: "1000"       # User ID
```

## 7. Rules

### Modification Rules

1. **Adding New Service**:
   - Add `use_<service>`, `<service>_url`, `<service>_domain` to `defaults/main.yml`
   - Create server block in `templates/default.conf.j2`
   - Wrap in `{% if use_<service> %}` conditional
   - Follow existing proxy pattern
   - Document in README.md

2. **Changing Defaults**:
   - Update `defaults/main.yml`
   - Do NOT modify templates directly for environment-specific values
   - Override in playbook vars or inventory

3. **Updating Templates**:
   - Test locally before committing
   - Ensure backward compatibility
   - Use `backup: true` in template task
   - Validate nginx syntax: `nginx -t`

4. **Security Changes**:
   - Always review proxy headers
   - Maintain security headers
   - Test SSL/TLS configurations
   - Never commit certificates to git

### Testing Rules

1. Run yamllint: `yamllint .`
2. Test syntax: `ansible-playbook --syntax-check tests/test.yml`
3. Dry run: `ansible-playbook --check tests/test.yml`
4. Validate nginx config: `docker exec nginx-dev nginx -t`
5. Check container logs: `docker logs nginx-dev`

### Version Control Rules

- Use semantic versioning
- Tag releases in git
- Document breaking changes
- Keep CHANGELOG.md updated
- Never commit sensitive data (.env files, certs, passwords)

### Performance Considerations

- Use `access_log off` for health checks
- Implement upstream for load balancing when needed
- Monitor container resource usage
- Rotate logs regularly
- Use nginx caching when appropriate

### Debugging

**Common Commands**:
```bash
# Check container status
docker ps | grep nginx

# View logs
docker logs nginx-dev

# Inspect config
docker exec nginx-dev cat /etc/nginx/conf.d/default.conf

# Test config syntax
docker exec nginx-dev nginx -t

# Reload config
docker exec nginx-dev nginx -s reload
```

**Common Issues**:
1. Port conflicts: Check with `netstat -tulpn | grep <port>`
2. Permission issues: Verify file ownership matches `adm_user:adm_group`
3. Template errors: Check Jinja2 syntax, validate variables exist
4. Container crashes: Check `docker logs` for startup errors
5. Proxy failures: Verify backend service URLs are accessible

---

**Last Updated**: 2025-11-01
**Role Version**: Based on current git commit
**Maintainer**: CodeSuiteApp
