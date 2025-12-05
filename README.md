# TheHive + Cortex Deployment - Critical Issues & Solutions

## Issue 1: TheHive Command Syntax Error
**Error:** `Unknown parameter '--secret=lab123456789'`
**Fix:** Change command format in docker-compose.yml:
```yaml
command:
  - --secret
  - thehive1234
  - --cql-hostnames  
  - cassandra
```

## Issue 2: Cassandra Port Conflict
**Error:** Port 9042 already in use
**Fix:** Use different external port, internal connection remains same:
```yaml
ports:
  - "9043:9042"
# TheHive connects via service name 'cassandra' internally
```

## Issue 3: Cortex Authentication Error
**Error:** `unrecognized option: --es-username`
**Fix:** Disable Elasticsearch security in docker-compose.yml:
```yaml
environment:
  - "xpack.security.enabled=false"
```

## Issue 4: Analyzers Not Visible in UI
**Error:** Logs show analyzers loaded but UI empty
**Root Cause:** Organization setup incomplete + manual refresh needed

**Fix Steps:**
1. **Create New Organization** (not "default")
2. **Create orgAdmin user** in that organization  
3. **Login as orgAdmin**
4. **Go to Organization → Analyzers**
5. **Click "Refresh analyzers" button**
6. **Repeat for Responders tab**

## Issue 5: TheHive-Cortex Integration
**Error:** AUTH_ERROR connecting to Cortex
**Fix:** 
1. Generate API key in Cortex
2. Add to TheHive docker-compose:
```yaml
command:
  - --cortex-port
  - "9001"
  - --cortex-keys
  - "YOUR_API_KEY_HERE"
```
3. Restart TheHive container

## Critical Configuration Files

### cortex-application.conf
```
analyzer {
  urls = ["https://download.thehive-project.org/analyzers.json"]
}
responder {
  urls = ["https://download.thehive-project.org/responders.json"]  
}
job {
  runner = [docker]
}
```

### Docker Socket Permissions
```bash
chmod 666 /var/run/docker.sock
```

## Verification Commands
```bash
# Check analyzers loaded
docker-compose logs cortex | grep -i "analyzer\|catalog"

# Expected output includes:
# DevTools_Echo_Analyzer 1.0
# Crowdsec_Analyzer 1.1
# TestAnalyzer 1.0
```

## Final Access URLs
- **TheHive:** http://localhost:9000 (admin@thehive.local / secret)
- **Cortex:** http://localhost:9001 (admin / thehive) 

## Key Learning Points
1. **Organization must be created manually** - default doesn't work
2. **Manual refresh required** for analyzers/responders visibility
3. **Cortex 3.1.1 more stable** than 3.1.7 for analyzer integration
4. **Elasticsearch security must be disabled** for Cortex 3.1.x
5. **API key integration essential** for TheHive-Cortex communication

## Test Integration
1. Create case in TheHive
2. Add observable (IP/domain/hash)
3. Run analyzers - should see Cortex analyzers available
4. View analysis results
