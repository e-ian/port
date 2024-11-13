# YouTube Playlist & Videos catalogue in Port

A comprehensive guide for creating YouTube playlist & Videos catalogue in Port, including data modeling, ingestion, and visualization.

## Overview

This guide shows you how to set up a system for tracking YouTube playlists and video performance in Port. 
You’ll learn how to organize your YouTube content, automate the tracking of video metrics, and create visualizations 
to monitor playlist growth and audience engagement. The integration provides centralized content management, 
real-time performance updates, and automated daily data updates, giving you the insights needed to make informed content decisions.

## Prerequisites

Before starting this integration, ensure you have:
   - Sign up at [app.getport.io](https://app.getport.io)
   - Log into your Port account at [app.getport.io](https://app.getport.io)
   - Install Port's Github app by clicking [here](https://github.com/apps/getport-io/installations/new)
   - Obtain a YouTube Data API v3 key via the [Google Cloud Console](https://console.cloud.google.com)
   - Prepare your Port organization's Client ID and Client Secret. To find you Port credentials, click [here](https://docs.getport.io/build-your-software-catalog/custom-integration/api/#find-your-port-credentials).

## Data Modeling in Port
This section outlines the data model for managing YouTube content in Port. The model consists of two main blueprints: `playlist` and `video`, designed to track YouTube playlists and their associated videos. The relationship between playlists and videos is maintained through a one-to-many relationship, where each video belongs to a playlist.

### Setting up Blueprints
   - Log into Port
   - Click ["Builder"](https://app.getport.io/settings/data-model) in the left sidebar
   - Click "+ New Blueprint"
   - In the model that pops up, click on "{---} Edit JSON"
   - Copy and paste the json configs below to have for the Playlist and Video blueprint properties


<details>
<summary>Click to expand Playlist Blueprint JSON</summary>

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

</details>


<details>
<summary>Click to expand Video Blueprint JSON</summary>

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

</details>

## Self service setup
  - Navigate to Self-service in Port by clicking [here](https://app.getport.io/self-serve)
  - Click on the "+ Action" button to create and new action. Click on "{---} Edit JSON" in the pop up
  - Copy and paste the json configuration details below to setup the Self-Service. Edit with your details.

  <details>
  <summary>Click to expand Self-service config JSON</summary>

  ```json
  {
  "identifier": "youtube",
  "title": "youtube",
  "icon": "Github",
  "description": "Self service action to trigger an action that fetches a youtube playlist",
  "trigger": {
    "type": "self-service",
    "operation": "CREATE",
    "userInputs": {
      "properties": {
        "playlist_id": {
          "icon": "Youtrack",
          "type": "string",
          "title": "playlist_id",
          "description": "Playlist id to be used for fetching the different respective videos"
        }
      },
      "required": [
        "playlist_id"
      ],
      "order": [
        "playlist_id"
      ]
    },
    "blueprintIdentifier": "playlist"
  },
  "invocationMethod": {
    "type": "GITHUB",
    "org": "<YOUR_ORGANISATION_NAME>",
    "repo": "<YOUR_REPO_NAME>",
    "workflow": "<SPECIFY_WORKFLOW_FILE>",
    "workflowInputs": {
      "{{ spreadValue() }}": "{{ .inputs }}",
      "port_context": {
        "runId": "{{ .run.id }}",
        "blueprint": "{{ .action.blueprint }}"
      }
    },
    "reportWorkflowStatus": true
  },
  "requiredApproval": false
}

  ```

  </details>

## GitHub workflow
   - Go to your GitHub repository settings
   - Add each of these secrets:
     ```
     YOUTUBE_API_KEY (Your YouTube Data API key)
     PORT_API_KEY (Your Port API key)
     PORT_CLIENT_ID (Your Port Client ID)
     PORT_CLIENT_SECRET (Your Port Client Secret)
     ```
   - Create new file: `ingest.yml` in the directory `.github/workflows` and pass the secrets.
   - Add the following GitHub Actions workflow configuration:

<details>
<summary>Click to expand GitHub Workflow YAML</summary>

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

</details>

## Example self-service action
- On the Self-service tab, navigate to the self-service you created and click on "Create".
- The pop-up below will be shown, enter the value of the youtube `playlist_id` and click execute
<details>
<summary>Click to see example Self-Service Execution</summary>
<img src="./assets/execute.png" alt="Self-Service Execution">
</details>
- This will trigger the workflow and you can track the progress from your port account.
- When completed, go to catalogue to view the playlist and videos respectively.


## Creating Visualizations

After the data is ingested, create these visualizations to monitor your YouTube content:

### 1. Video Count Metric
1. Navigate to "Dashboards" in Port
2. Click "+ Add Widget"
3. Select "Number Chart"
4. Configure:

<details>
<summary>Click to see details and example configuration image</summary>
<img src="./assets/videocount.png" alt="Video count in playlist">
</details>

### 2. Average Likes Metric
1. Click "+ Add Widget"
2. Select "Number Chart"
3. Configure:

<details>
<summary>Click to see details and example configuration image</summary>
<img src="./assets/averagelikes.png" alt="Average likes card">
</details>

### 3. Video Details Table
1. Click "+ Add Widget"
2. Select "Table"
3. Configure:

<details>
<summary>Click to see details and example configuration image</summary>
<img src="./assets/videodetails.png" alt="Video details table">
</details>

### 4. Engagement Distribution Pie Chart
1. Click "+ Add Widget"
2. Select "Pie Chart"
3. Configure:

<details>
<summary>Click to see details and example configuration image</summary>
<img src="./assets/engagementdist.png" alt="Video engagement distribution">
</details>

### YouTube Playlist Analytics Dashboard Layout

<details>
<summary>Click to see example YouTube Analytics Dashboard</summary>
<img src="./assets/youtube_analytics_dashboard.png" alt="YouTube Analytics Dashboard">
</details>
