# Threadfin IPTV Proxy with EPG

## Overview

This project sets up Threadfin, an IPTV proxy for Plex, along with an automatic EPG (Electronic Program Guide) grabber. Threadfin emulates an HDHomeRun tuner, allowing Plex to integrate IPTV streams seamlessly.

### What's Included
- Threadfin: Active fork of xTeVe that acts as an M3U proxy for Plex/Emby
- EPG Grabber: Automatically fetches program guide data from iptv-org/epg

---

## Quick Start

### Prerequisites
- Docker and Docker Compose installed
- An IPTV subscription with an M3U URL

### Installation

1. Create the project folder:
   mkdir threadfin-epg && cd threadfin-epg

2. Create the docker-compose.yml file with this content:

   services:
     threadfin:
       image: fyb3roptik/threadfin:1.2.16
       container_name: threadfin
       restart: unless-stopped
       ports:
         - "34400:34400"
       environment:
         - PUID=1000
         - PGID=1000
         - TZ=Europe/Madrid
       volumes:
         - ./threadfin/config:/home/threadfin/conf
         - ./threadfin/temp:/tmp/threadfin:rw

     epg:
       image: ghcr.io/iptv-org/epg:master
       container_name: epg
       restart: unless-stopped
       ports:
         - "3001:3000"
       environment:
         - CRON_SCHEDULE=0 5 * * *
         - RUN_AT_STARTUP=true
       volumes:
         - ./epg/channels.xml:/epg/public/channels.xml:ro

3. Start the containers:
   docker-compose up -d

4. Verify they're running:
   docker-compose ps

---

## Configuration

### Step 1: Configure Threadfin

1. Open your browser and go to http://YOUR_PC_IP:34400
2. Navigate to Settings (gear icon)
3. Add your IPTV source:
   - Go to the M3U section and add your provider's M3U URL
   - Click Save
4. Add the EPG source:
   - Go to the XMLTV section
   - Add a new source with URL: http://YOUR_PC_IP:3001/guide.xml
   - Click Save

### Step 2: Channel Mapping

1. Go to the Channels tab
2. Threadfin will automatically attempt to match channels to guide data
3. For channels not matched automatically:
   - Click Edit Mapping on each channel
   - Find the correct guide channel from the dropdown
   - Click Save

### Step 3: Connect to Plex

1. In Plex, go to Settings > Live TV & DVR
2. Click Add Tuner and select HDHomeRun
3. Plex should auto-detect the Threadfin tuner
4. Follow the on-screen prompts to map your channels and set up the guide

---

## Customization

### EPG Configuration

To customize which channels get guide data:

1. Create or edit the channels file:
   nano ./epg/channels.xml

2. Add your channels using this format:

   <channels>
     <channel id="channel1.example.com">
       <display-name>BBC One</display-name>
     </channel>
     <channel id="channel2.example.com">
       <display-name>CNN</display-name>
     </channel>
   </channels>

3. Restart the EPG container:
   docker-compose restart epg

### Advanced Threadfin Settings

Uncomment these environment variables in docker-compose.yml for more control:

   environment:
     - THREADFIN_DEBUG=1      # Enable debug logging (0-3)
     - THREADFIN_PORT=34400   # Change web UI port
     - THREADFIN_BRANCH=beta  # Use beta version

---

## Troubleshooting

### 1. EPG Not Loading

Check the EPG log:
   docker-compose logs epg

Verify the EPG is accessible:
   curl http://localhost:3001/guide.xml

Manually trigger an EPG update:
   docker-compose exec epg node /epg/index.js

### 2. Threadfin Not Connecting to Plex

Check Threadfin logs:
   docker-compose logs threadfin

Verify Threadfin is accessible:
   curl http://localhost:34400

Restart Threadfin:
   docker-compose restart threadfin

### 3. Channel Mapping Issues

Reload the XMLTV data:
1. In Threadfin web UI, go to XMLTV settings
2. Click Reload or Force Update
3. Wait for the reload to complete

---

## Maintenance

### Update Containers
   docker-compose pull
   docker-compose up -d

### Backup Configuration
   # Backup Threadfin settings
   tar -czf threadfin-backup-$(date +%Y%m%d).tar.gz ./threadfin/

   # Backup EPG channels
   tar -czf epg-backup-$(date +%Y%m%d).tar.gz ./epg/

### Stop Containers
   docker-compose down

### Stop and Remove Everything
   docker-compose down -v

---

## File Structure

After running the containers, your folder will look like this:

   threadfin-epg/
   ├── docker-compose.yml
   ├── threadfin/
   │   ├── config/
   │   └── temp/
   └── epg/
       └── channels.xml

---

## Resources

- Threadfin GitHub: https://github.com/Threadfin/Threadfin
- IPTV-org EPG: https://github.com/iptv-org/epg
- Plex Live TV & DVR: https://support.plex.tv/articles/225877427-live-tv-dvr/