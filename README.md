# 📢 Jellyfin → Discord Webhook Integration
---

## ✨ Features

### 🎯 Multi-Event Support

* New item additions: **Movies, Episodes, Seasons, Albums, Songs**
* **Playback events**: Start / Stop (with progress)
* **User events**: Lockout notifications

### 🎨 Visual Enhancements

* Emoji icons for different notification types
* Color-coded embeds:

  * 🟢 Green → Item additions, playback completions
  * 🔵 Blue → Playback starts
  * 🔴 Red → Playback stops & user lockouts
* Clean, organized embed structure

### 📊 Rich Information Display

* Media details: Title, year, runtime, genres
* External links:

  * IMDb, TMDb, TVMaze, MusicBrainz, AudioDb
* User & device info for playback events
* Direct **Jellyfin Web UI links** for items

### 🔧 Smart Logic

* Conditional formatting based on:

  * Notification type (additions, playback, user events)
  * Media type (Movie, Series, Season, Episode, Music)
* Automatic episode/season formatting (`S01E01`, etc.)
* Completion status detection for playback stops
* Different avatars for playback vs server events

---

## 📂 Setup Guide

### 1. Install the Jellyfin Webhook Plugin

* In Jellyfin, go to **Dashboard → Plugins → Catalog**
* Install the **Webhook** plugin.
* Restart Jellyfin.

### 2. Save the Template to your system

* Copy or download the provided Discord webhook template (see **Discord Webhook Template** below).

### 3. Configure the Webhook in Jellyfin

1. Go to **Dashboard → Webhooks** in your Jellyfin server.
2. Click **Add** (or **Create New Webhook**).
3. **Destination**: paste your Discord webhook URL.

   * To create a Discord webhook: in your Discord server go to **Server Settings → Integrations → Webhooks → New Webhook**. Copy the webhook **URL**.
4. **Template**: * Copy and Paste the JSON from step 2 to template field in Webhooks section. (`discord-webhook-template.json`).
5. Optionally set:
   * Which notification types to send (e.g., ItemAdded, PlaybackStart, PlaybackStop, UserLockout).
   * 
---
---

## 📄 Discord Webhook Template

```json
{
    "content": "{{MentionType}}",
    "avatar_url": "{{#if_equals NotificationType 'PlaybackStart'}}{{ServerUrl}}/Users/{{UserId}}/Images/Primary{{else}}{{#if_equals NotificationType 'PlaybackStop'}}{{ServerUrl}}/Users/{{UserId}}/Images/Primary{{else}}{{AvatarUrl}}{{/if_equals}}{{/if_equals}}",
    "username": "{{BotUsername}}",
    "embeds": [
        {
            "author": {
                {{#if_equals NotificationType 'ItemAdded'}}
                    {{#if_equals ItemType 'Season'}}
                        "name": "Season Added • {{{SeriesName}}} {{{Name}}}",
                    {{else}}
                        {{#if_equals ItemType 'Episode'}}
                            "name": "Episode Added • {{{SeriesName}}} S{{SeasonNumber00}}E{{EpisodeNumber00}} ~ {{{Name}}}",
                        {{else}}
                            {{#if_equals ItemType 'Movie'}}
                                "name": "Movie Added • {{{Name}}} ({{Year}})",
                            {{else}}
                                {{#if_equals ItemType 'Album'}}
                                    "name": "Album Added • {{{Artist}}} - {{{Name}}}",
                                {{else}}
                                    {{#if_equals ItemType 'Audio'}}
                                        "name": "Song Added • {{{Artist}}} - {{{Name}}}",
                                    {{else}}
                                        "name": "Item Added • {{{Name}}}",
                                    {{/if_equals}}
                                {{/if_equals}}
                            {{/if_equals}}
                        {{/if_equals}}
                    {{/if_equals}}
                {{else}}
                    {{#if_equals NotificationType 'PlaybackStart'}}
                        {{#if_equals ItemType 'Episode'}}
                            "name": "Playback Started • {{{SeriesName}}} S{{SeasonNumber00}}E{{EpisodeNumber00}} ~ {{{Name}}}",
                        {{else}}
                            "name": "Playback Started • {{{Name}}} ({{Year}})",
                        {{/if_equals}}
                    {{else}}
                        {{#if_equals NotificationType 'PlaybackStop'}}
                            {{#if_equals ItemType 'Episode'}}
                                {{#if_equals PlayedToCompletion true}}
                                    "name": "Playback Completed • {{{SeriesName}}} S{{SeasonNumber00}}E{{EpisodeNumber00}} ~ {{{Name}}}",
                                {{else}}
                                    "name": "Playback Stopped • {{{SeriesName}}} S{{SeasonNumber00}}E{{EpisodeNumber00}} ~ {{{Name}}}",
                                {{/if_equals}}
                            {{else}}
                                {{#if_equals PlayedToCompletion true}}
                                    "name": "Playback Completed • {{{Name}}} ({{Year}})",
                                {{else}}
                                    "name": "Playback Stopped • {{{Name}}} ({{Year}})",
                                {{/if_equals}}
                            {{/if_equals}}
                        {{else}}
                            {{#if_equals NotificationType 'UserLockedOut'}}
                                "name": "User Locked Out • {{{NotificationUsername}}}",
                            {{else}}
                                "name": "Notification • {{NotificationType}}",
                            {{/if_equals}}
                        {{/if_equals}}
                    {{/if_equals}}
                {{/if_equals}}
                "url": "{{#if_equals NotificationType 'UserLockedOut'}}{{ServerUrl}}/web/index.html#!/useredit.html?userId={{UserId}}{{else}}{{ServerUrl}}/web/index.html#!/details?id={{ItemId}}&serverId={{ServerId}}{{/if_equals}}"
            },
            "thumbnail": {
                {{#if_equals NotificationType 'UserLockedOut'}}
                    "url": "{{ServerUrl}}/Users/{{UserId}}/Images/Primary"
                {{else}}
                    "url": "{{ServerUrl}}/Items/{{ItemId}}/Images/Primary"
                {{/if_equals}}
            },
            "description": "{{#if_equals NotificationType 'UserLockedOut'}}> {{{NotificationUsername}}} has been locked out due to too many failed login attempts.{{else}}{{#if_exist Overview}}> {{{Overview}}}{{/if_exist}}{{/if_equals}}{{#if_equals NotificationType 'PlaybackStart'}}{{#if_exist Overview}}\n\n{{/if_exist}}``[{{PlaybackPosition}}/{{RunTime}}]``{{/if_equals}}{{#if_equals NotificationType 'PlaybackStop'}}{{#if_exist Overview}}\n\n{{/if_exist}}``[{{PlaybackPosition}}/{{RunTime}}]``{{/if_equals}}{{#if_equals NotificationType 'ItemAdded'}}{{#if_exist ServerUrl}}\n\n[**Watch Now**]({{ServerUrl}}/web/index.html#!/details?id={{ItemId}}&serverId={{ServerId}}){{/if_exist}}{{/if_equals}}{{#if_exist Provider_imdb}} • [**IMDb**](https://www.imdb.com/title/{{Provider_imdb}}/){{/if_exist}}{{#if_exist Provider_tmdb}}{{#if_equals ItemType 'Movie'}} • [**TMDb**](https://www.themoviedb.org/movie/{{Provider_tmdb}}){{else}} • [**TMDb**](https://www.themoviedb.org/tv/{{Provider_tmdb}}){{/if_equals}}{{/if_exist}}{{#if_exist Provider_tvmaze}}{{#if_equals ItemType 'Episode'}} • [**TVMaze**](https://www.tvmaze.com/episodes/{{Provider_tvmaze}}){{/if_equals}}{{#if_equals ItemType 'Series'}} • [**TVMaze**](https://www.tvmaze.com/shows/{{Provider_tvmaze}}){{/if_equals}}{{/if_exist}}",
            {{#if_equals NotificationType 'ItemAdded'}}
                "color": 3381759,
            {{else}}
                {{#if_equals NotificationType 'PlaybackStart'}}
                    "color": 3394611,
                {{else}}
                    {{#if_equals NotificationType 'PlaybackStop'}}
                        {{#if_equals PlayedToCompletion true}}
                            "color": 3381759,
                        {{else}}
                            "color": 15737923,
                        {{/if_equals}}
                    {{else}}
                        "color": 15737923,
                    {{/if_equals}}
                {{/if_equals}}
            {{/if_equals}}
            "footer": {
                "text": "{{{ServerName}}}",
                "icon_url": "{{AvatarUrl}}"
            },
            {{#if_equals NotificationType 'PlaybackStart'}}
                "fields": [
                    {
                        "name": "User",
                        "value": "{{{NotificationUsername}}}",
                        "inline": true
                    }
                ],
            {{else}}
                {{#if_equals NotificationType 'PlaybackStop'}}
                    "fields": [
                        {
                            "name": "User",
                            "value": "{{{NotificationUsername}}}",
                            "inline": true
                        }
                    ],
                {{else}}
                    "fields": [],
                {{/if_equals}}
            {{/if_equals}}
            "timestamp": "{{UtcTimestamp}}"
        }
    ]
}
```

> **Note:** The template uses Jellyfin’s handlebars-like conditionals such as `if_equals` and `if_exist`. Although the file looks like JSON, it's a template processed by the Jellyfin webhook plugin — keep the handlebars blocks exactly as shown.

---
