---
title: Troubleshooting
description: Common issues and solutions
---

# Troubleshooting

## Container Can't Reach Internet

**Symptom**: Clawdbot can't connect to WhatsApp/Telegram

**Check**:
```bash
# Test from container
sudo docker exec clawdbot ping -c 3 8.8.8.8

# Check UFW allows outbound
sudo ufw status verbose | grep OUT
```

**Solution**:
```bash
# Verify DOCKER-USER allows established connections
sudo iptables -L DOCKER-USER -n -v

# Restart Docker + Firewall
sudo systemctl restart docker
sudo ufw reload
sudo systemctl restart clawdbot
```

## Port Already in Use

**Symptom**: Port 3000 conflict

**Solution**:
```bash
# Find what's using port 3000
sudo ss -tlnp | grep 3000

# Change Clawdbot port
sudo nano /opt/clawdbot/docker-compose.yml
# Change: "127.0.0.1:3001:3000"

sudo systemctl restart clawdbot
```

## Firewall Lockout

**Symptom**: Can't SSH after installation

**Solution** (via console/rescue mode):
```bash
# Disable UFW temporarily
sudo ufw disable

# Check SSH rule exists
sudo ufw status numbered

# Re-add SSH rule
sudo ufw allow 22/tcp

# Re-enable
sudo ufw enable
```

## Container Won't Start

**Check logs**:
```bash
# Systemd logs
sudo journalctl -u clawdbot -n 50

# Docker logs
sudo docker logs clawdbot

# Compose status
sudo docker compose -f /opt/clawdbot/docker-compose.yml ps
```

**Common fixes**:
```bash
# Rebuild image
cd /opt/clawdbot
sudo docker compose build --no-cache
sudo systemctl restart clawdbot

# Check permissions
sudo chown -R clawdbot:clawdbot /home/clawdbot/.clawdbot
```

## Verify Docker Isolation

**Test that external ports are blocked**:
```bash
# Start test container
sudo docker run -d -p 80:80 --name test-nginx nginx

# From EXTERNAL machine (should fail):
curl http://YOUR_SERVER_IP:80

# From SERVER (should work):
curl http://localhost:80

# Cleanup
sudo docker rm -f test-nginx
```

## UFW Status Shows Inactive

**Fix**:
```bash
# Enable UFW
sudo ufw enable

# Reload rules
sudo ufw reload

# Verify
sudo ufw status verbose
```

## Ansible Playbook Fails

**Collection missing**:
```bash
ansible-galaxy collection install -r requirements.yml
```

**Permission denied**:
```bash
# Run with --ask-become-pass
ansible-playbook playbook.yml --ask-become-pass
```

**Docker daemon not running**:
```bash
sudo systemctl start docker
# Re-run playbook
```
