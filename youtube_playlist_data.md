# YouTube Playlist & Videos catalogue in Port

A comprehensive guide for creating YouTube playlist & Videos catalogue in Port, including data modeling, ingestion, and visualization.

## Overview

This guide shows you how to set up a system for tracking YouTube playlists and video performance in Port. 
You’ll learn how to organize your YouTube content, automate the tracking of video metrics, and create visualizations 
to monitor playlist growth and audience engagement. The integration provides centralized content management, 
real-time performance updates, and automated daily data updates, giving you the insights needed to make informed content decisions.

## Prerequisites

Before starting this integration, ensure you have:

1. **Port Account**
   - Sign up at [app.getport.io](https://app.getport.io)
   - Verify your email and log in

   **Installing Port's GitHub App**
   - Log into your Port account at [app.getport.io](https://app.getport.io)
   - Navigate to "Data Sources" in the left sidebar
   - Find and click on "GitHub" under the available integrations
   - Click "Install GitHub App"
   - You will be redirected to GitHub's app installation page
   - Choose whether to install the app on all repositories or select specific ones
   - If selecting specific repositories:
     - Choose the repositories you want to monitor
     - Click "Install & Authorize"
   - Return to Port's "Data Sources" page
   - Check that GitHub appears as a connected integration
   - The status should show as "Active"


2. **GitHub Account**
   - Active GitHub account
   - Repository where you'll set up the workflow

3. **API Keys and Credentials**
   - YouTube Data API v3 key
     - Go to [Google Cloud Console](https://console.cloud.google.com)
     - Create a project or select existing one
     - Enable YouTube Data API v3
     - Create API credentials
   - Port API key
   - Port Client ID and Secret

## Part 1: Data Modeling in Port

### Creating the Playlist Blueprint

1. **Navigate to Builder**
   - Log into Port
   - Click "Builder" in the left sidebar
   - Click "+ New Blueprint"

2. **Configure Basic Settings**
   - Name: "playlist"
   - Display Name: "Playlist"
   - Description: "YouTube playlist description"
   - Icon: Select "Microservice" from the dropdown

3. **Add Properties**
   - Click "Add Property"
   - Add the following properties:

   a) Title Property
   - Name: "title"
   - Type: "String"
   - Required: Yes
   - Description: "Title of the playlist"
   
   b) Description Property
   - Name: "description"
   - Type: "String"
   - Required: No
   - Description: "The description of the playlist"
   
   c) Thumbnail URL Property
   - Name: "thumbnail_url"
   - Type: "String"
   - Format: "URL"
   - Required: No
   - Description: "The URL of the playlist's thumbnail image"
   
   d) Video Count Property
   - Name: "video_count"
   - Type: "Number"
   - Required: No
   - Description: "The number of videos in the playlist"

4. **Save Blueprint**
   - Click "Create" at the bottom of the page
   - Verify the blueprint appears in your blueprint list

### Creating the Video Blueprint

1. **Start New Blueprint**
   - Click "+ New Blueprint"
   - Name: "video"
   - Display Name: "Video"
   - Description: "YouTube video blueprint"
   - Icon: Select "Microservice"

2. **Add Properties**
   - Add the following properties:

   a) Title Property
   - Name: "title"
   - Type: "String"
   - Required: Yes
   - Description: "The title of the video"
   
   b) Description Property
   - Name: "description"
   - Type: "String"
   - Required: No
   - Description: "The description of the video"
   
   c) Thumbnail URL Property
   - Name: "thumbnail_url"
   - Type: "String"
   - Format: "URL"
   - Required: No
   - Description: "The URL of the video's thumbnail image"
   
   d) Duration Property
   - Name: "duration"
   - Type: "String"
   - Required: No
   - Description: "The duration of the video"
   
   e) View Count Property
   - Name: "view_count"
   - Type: "Number"
   - Required: No
   - Description: "The number of views the video has received"
   
   f) Like Count Property
   - Name: "like_count"
   - Type: "Number"
   - Required: No
   - Description: "The number of likes the video has received"
   
   g) Comment Count Property
   - Name: "comment_count"
   - Type: "Number"
   - Required: No
   - Description: "The number of comments the video has received"

3. **Add Relation**
   - Click "Add Relation"
   - Name: "belongs_to"
   - Title: "Belongs To"
   - Description: "Relationship between video and playlist"
   - Target Blueprint: Select "playlist"
   - Required: Yes
   - Many: No

4. **Save Blueprint**
   - Click "Create"
   - Verify the blueprint appears with all properties and relations

### Blueprint YAML Representations

When creating the blueprints in Port, you can also use these YAML definitions:

#### Playlist Blueprint YAML

```json
{
  "identifier": "playlist",
  "description": "Youtube playlist description",
  "title": "playlist",
  "icon": "Microservice",
  "schema": {
    "properties": {
      "title": {
        "type": "string",
        "title": "title",
        "description": "title of the playlist"
      },
      "description": {
        "type": "string",
        "title": "description",
        "description": "the description of the playlist"
      },
      "thumbnail_url": {
        "type": "string",
        "title": "thumbnailUrl",
        "description": "the URL of the playlist's thumbnail image",
        "format": "url"
      },
      "video_count": {
        "type": "number",
        "title": "videoCount",
        "description": "The number of videos in the playlist"
      }
    },
    "required": ["title"]
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "aggregationProperties": {},
  "relations": {}
}
```

#### Video Blueprint YAML
```json
{
  "identifier": "video",
  "description": "youtube video blueprint",
  "title": "video",
  "icon": "Microservice",
  "schema": {
    "properties": {
      "title": {
        "type": "string",
        "title": "title",
        "description": "the title of the video"
      },
      "description": {
        "type": "string",
        "title": "description",
        "description": "the description of the video"
      },
      "thumbnail_url": {
        "type": "string",
        "title": "thumbnailUrl",
        "description": "The URL of the video's thumbnail image",
        "format": "url"
      },
      "duration": {
        "type": "string",
        "title": "duration",
        "description": "the duration of the video"
      },
      "view_count": {
        "type": "number",
        "title": "viewCount",
        "description": "The number of views the video has received"
      },
      "like_count": {
        "type": "number",
        "title": "likeCount",
        "description": "The number of likes the video has received"
      },
      "comment_count": {
        "type": "number",
        "title": "commentCount",
        "description": "The number of comments the video has received"
      }
    },
    "required": ["title"]
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "aggregationProperties": {},
  "relations": {
    "belongs_to": {
      "title": "Belongs To",
      "description": "relationship between video and playlist",
      "target": "playlist",
      "required": true,
      "many": false
    }
  }
}
```

## Part 2: GitHub Workflow Setup

### Setting Up Repository Secrets

1. **Navigate to Repository Settings**
   - Go to your GitHub repository
   - Click "Settings"
   - Click "Secrets and variables" in the left sidebar
   - Select "Actions"

2. **Add Required Secrets**
   - Click "New repository secret"
   - Add each of these secrets:
     ```
     YOUTUBE_API_KEY (Your YouTube Data API key)
     PORT_API_KEY (Your Port API key)
     PORT_CLIENT_ID (Your Port Client ID)
     PORT_CLIENT_SECRET (Your Port Client Secret)
     ```

### Creating the Workflow File

1. **Create Workflow Directory**
   - In your repository, create directory: `.github/workflows`

2. **Create Workflow File**
   - Create new file: `ingest.yml`
   - Add the following GitHub Actions workflow configuration:

```yaml
name: Ingest YouTube Playlist

on:
  workflow_dispatch:
    inputs:
      playlist_id:
        description: "Youtube video playlist id"
        required: true
      port_context:
        description: "The port context"
        required: true

jobs:
  ingest-data:
    runs-on: ubuntu-latest
    env:
      YOUTUBE_API_KEY: ${{ secrets.YOUTUBE_API_KEY }}
      PORT_CLIENT_ID: ${{ secrets.PORT_CLIENT_ID }}
      PORT_CLIENT_SECRET: ${{ secrets.PORT_CLIENT_SECRET }}
      PLAYLIST_ID: ${{ github.event.inputs.playlist_id }}
    steps:
      - name: Process Playlist and Videos
        run: |
          # Get Port access token
          echo "Getting Port access token"
          TOKEN_RESPONSE=$(curl -s -X POST "https://api.getport.io/v1/auth/access_token" \
            -H "Content-Type: application/json" \
            -d "{
              \"clientId\": \"${PORT_CLIENT_ID}\",
              \"clientSecret\": \"${PORT_CLIENT_SECRET}\"
            }")
          
          PORT_TOKEN=$(echo $TOKEN_RESPONSE | jq -r '.accessToken')
          if [ -z "$PORT_TOKEN" ] || [ "$PORT_TOKEN" = "null" ]; then
            echo "Failed to get access token"
            echo "Response: $TOKEN_RESPONSE"
            exit 1
          fi
          
          # Function to create Port entity
          create_port_entity() {
            local BLUEPRINT=$1
            local PAYLOAD=$2
            curl -s -X POST "https://api.getport.io/v1/blueprints/${BLUEPRINT}/entities" \
              -H "Authorization: Bearer ${PORT_TOKEN}" \
              -H "Content-Type: application/json" \
              -d "$PAYLOAD"
          }

          echo "Fetching playlist data"
          PLAYLIST_DATA=$(curl -s "https://youtube.googleapis.com/youtube/v3/playlists?part=snippet,contentDetails&id=${PLAYLIST_ID}&key=${YOUTUBE_API_KEY}")
          
          if [ "$(echo $PLAYLIST_DATA | jq '.items | length')" -eq 0 ]; then
            echo "Error: No playlist found"
            exit 1
          fi

          # Process playlist
          TITLE=$(echo $PLAYLIST_DATA | jq -r '.items[0].snippet.title')
          DESC=$(echo $PLAYLIST_DATA | jq -r '.items[0].snippet.description')
          THUMB=$(echo $PLAYLIST_DATA | jq -r '.items[0].snippet.thumbnails.default.url')
          COUNT=$(echo $PLAYLIST_DATA | jq -r '.items[0].contentDetails.itemCount')

          # Create sanitized JSON for playlist
          PLAYLIST_PAYLOAD=$(jq -n \
            --arg id "$PLAYLIST_ID" \
            --arg title "$TITLE" \
            --arg desc "$DESC" \
            --arg thumb "$THUMB" \
            --arg count "$COUNT" \
            '{
              identifier: $id,
              title: $title,
              properties: {
                title: $title,
                description: $desc,
                thumbnail_url: $thumb,
                video_count: ($count|tonumber)
              }
            }')

          echo "Creating playlist entity"
          PLAYLIST_RESPONSE=$(create_port_entity "playlist" "$PLAYLIST_PAYLOAD")
          echo "Playlist Response: ${PLAYLIST_RESPONSE}"

          # Process videos
          process_videos() {
            local PAGE_TOKEN=$1
            local API_URL="https://youtube.googleapis.com/youtube/v3/playlistItems?part=contentDetails&maxResults=50&playlistId=${PLAYLIST_ID}&key=${YOUTUBE_API_KEY}"
            if [ -n "${PAGE_TOKEN}" ]; then
              API_URL="${API_URL}&pageToken=${PAGE_TOKEN}"
            fi

            local ITEMS_RESPONSE=$(curl -s "${API_URL}")
            echo $ITEMS_RESPONSE | jq -r '.items[].contentDetails.videoId' | while read -r VIDEO_ID; do
              echo "Processing video: ${VIDEO_ID}"
              
              VIDEO_DATA=$(curl -s "https://youtube.googleapis.com/youtube/v3/videos?part=snippet,contentDetails,statistics&id=${VIDEO_ID}&key=${YOUTUBE_API_KEY}")
              
              if [ "$(echo $VIDEO_DATA | jq '.items | length')" -gt 0 ]; then
                local V_TITLE=$(echo $VIDEO_DATA | jq -r '.items[0].snippet.title')
                local V_DESC=$(echo $VIDEO_DATA | jq -r '.items[0].snippet.description')
                local V_THUMB=$(echo $VIDEO_DATA | jq -r '.items[0].snippet.thumbnails.default.url')
                local V_DURATION=$(echo $VIDEO_DATA | jq -r '.items[0].contentDetails.duration')
                local V_VIEWS=$(echo $VIDEO_DATA | jq -r '.items[0].statistics.viewCount // "0"')
                local V_LIKES=$(echo $VIDEO_DATA | jq -r '.items[0].statistics.likeCount // "0"')
                local V_COMMENTS=$(echo $VIDEO_DATA | jq -r '.items[0].statistics.commentCount // "0"')

                # Create sanitized JSON for video
                local VIDEO_PAYLOAD=$(jq -n \
                  --arg id "$VIDEO_ID" \
                  --arg title "$V_TITLE" \
                  --arg desc "$V_DESC" \
                  --arg thumb "$V_THUMB" \
                  --arg duration "$V_DURATION" \
                  --arg views "$V_VIEWS" \
                  --arg likes "$V_LIKES" \
                  --arg comments "$V_COMMENTS" \
                  --arg playlist_id "$PLAYLIST_ID" \
                  '{
                    identifier: $id,
                    title: $title,
                    properties: {
                      title: $title,
                      description: $desc,
                      thumbnail_url: $thumb,
                      duration: $duration,
                      view_count: ($views|tonumber),
                      like_count: ($likes|tonumber),
                      comment_count: ($comments|tonumber)
                    },
                    relations: {
                      belongs_to: $playlist_id
                    }
                  }')

                VIDEO_RESPONSE=$(create_port_entity "video" "$VIDEO_PAYLOAD")
                echo "Video Response: ${VIDEO_RESPONSE}"
                sleep 1
              fi
            done

            local NEXT_PAGE=$(echo $ITEMS_RESPONSE | jq -r '.nextPageToken')
            if [ "${NEXT_PAGE}" != "null" ]; then
              process_videos "${NEXT_PAGE}"
            fi
          }

          echo "Starting video processing"
          process_videos ""
```


## Part 4: Creating Visualizations

After the data is ingested, create these visualizations to monitor your YouTube content:

### 1. Video Count Metric
1. Navigate to "Dashboards" in Port
2. Click "+ Add Widget"
3. Select "Number Chart"
4. Configure:
   - Title: "Total Videos"
   - Under Query Definition:
     - Blueprint: Select "video"
     - Aggregation: Select "Count"

### 2. Average Likes Metric
1. Click "+ Add Widget"
2. Select "Number Chart"
3. Configure:
   - Title: "Average Likes"
   - Under Query Definition:
     - Blueprint: Select "video"
     - Aggregation: Select "Average"
     - Property: Select "like_count"

### 3. Video Details Table
1. Click "+ Add Widget"
2. Select "Table"
3. Configure:
   - Title: "Video Details"
   - Select Blueprint: "video"
   - Add Columns:
     - Title
     - View Count
     - Like Count
     - Comment Count
     - Duration

### 4. Engagement Distribution Pie Chart
1. Click "+ Add Widget"
2. Select "Pie Chart"
3. Configure:
   - Title: "Video Engagement Distribution"
   - Select Blueprint: "video"
   - Group By: "view_count"

### YouTube Playlist Analytics Dashboard Layout

![YouTube Analytics Dashboard](./assets/youtube_analytics_dashboard.png)
