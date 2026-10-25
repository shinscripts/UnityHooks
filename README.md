# UnityHooks

**Send Discord webhooks directly from Unity using a straightforward C# API.**

UnityHooks is a lightweight wrapper for creating and sending Discord webhook messages from Unity projects. It lets you build rich Discord embeds without manually constructing JSON payloads or writing the HTTP request yourself.

Whether you're reporting a new high score, sharing a gameplay event, or posting server status updates, UnityHooks gives you two ways to compose messages: **method chaining** or **directly setting embed properties**.

[Project write-up and showcase](https://www.shahinmohseni.se/projects/UnityHooks)

## Features

- **Discord webhook integration** - Send messages to a Discord channel through its webhook URL.
- **Rich embeds** - Customize titles, descriptions, colors, fields, timestamps, authors, links, thumbnails, images, and footers.
- **Two configuration styles** - Use chainable methods or initialize the embed's public properties directly.
- **Asynchronous sending** - Send webhooks with `await` and inspect the result, response code, and error information.
- **JSON inspection** - Call `ToJson()` to inspect the generated webhook payload while developing or debugging.
- **Unity-focused** - Written in C# as a Unity project, with a practical demonstration described in the [project write-up](https://www.shahinmohseni.se/projects/UnityHooks).

## Getting started

### Requirements

- **Unity 6** - The repository was created with **Unity 6000.0.30f1**, not verified but should work with versions above this.
- **Newtonsoft JSON for Unity** - The project includes `com.unity.nuget.newtonsoft-json` version `3.2.1`.
- **A Discord webhook URL** - Create a webhook in the Discord server/channel where you want to post messages.

### 1. Open the project

Clone the repository:

```bash
git clone https://github.com/ShinScripts/UnityHooks.git
```

In Unity Hub, choose **Add project from disk**, select the cloned folder, and open it with the Unity version listed above.

If you want to use UnityHooks in an existing Unity project instead, bring over the relevant C# source files from this repository's `Assets` folder and make sure the required packages are available in that project.

### 2. Create a Discord webhook

In Discord, open the integration settings for the destination channel, create a webhook, and copy its URL. Discord's [webhook setup guide](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks) walks through the process.

The URL will look like this:

```text
https://discord.com/api/webhooks/WEBHOOK_ID/WEBHOOK_TOKEN
```

**Keep this URL private.** It contains a token that can be used to post to your channel. Do not commit a real webhook URL to Git, publish it in screenshots/logs, or embed it in a distributed game client. For publicly distributed games, send events to a trusted backend that holds the webhook URL securely.

### 3. Send your first embed

The following example uses the method-chaining API to send an embed with a title, color, and field. Place it inside an `async` C# method in your Unity project:

```csharp
using System;
using UnityEngine;

// Inside an async method:
Uri webhookUrl = new("https://discord.com/api/webhooks/WEBHOOK_ID/WEBHOOK_TOKEN");
Webhook hook = new(webhookUrl);

Embed embed = new Embed()
    .SetTitle("New high score!")
    .SetColor(Colors.Yellow)
    .AddField("Player", "PlayerOne")
    .AddField("Score", "12,500");

hook.AddEmbed(embed);

var (result, responseCode, error) = await hook.Send();

Debug.Log($"Result: {result}, Response code: {responseCode}, Error: {error}");
Debug.Log(hook.ToJson());
```

Replace the example URL with the webhook URL for your Discord channel, stored securely.

## Usage examples

### Option 1: Method chaining

Chain calls together when you want a concise and readable way to configure an embed:

```csharp
Embed embed = new Embed()
    .SetTitle("Server update")
    .SetColor(Colors.Yellow)
    .AddField("Status", "Online")
    .AddField("Players", "12/20");

Webhook hook = new(new Uri("https://discord.com/api/webhooks/WEBHOOK_ID/WEBHOOK_TOKEN"));
hook.AddEmbed(embed);

var (result, responseCode, error) = await hook.Send();
```

This is useful when you want to construct an embed step by step.

### Option 2: Direct property configuration

You can also work with an `Embed` object directly, including its nested fields and thumbnail:

```csharp
using System;
using System.Collections.Generic;

// Inside an async method:
Uri webhookUrl = new("https://discord.com/api/webhooks/WEBHOOK_ID/WEBHOOK_TOKEN");

Webhook hook = new(webhookUrl);

Embed embed = new()
{
    title = "Match summary",
    description = "A match has just finished.",
    url = new Uri("https://github.com/ShinScripts/UnityHooks"),
    color = Colors.Orange,
    timestamp = DateTime.UtcNow.ToString(),
    thumbnail = new()
    {
        url = new Uri("https://example.com/thumbnail.png")
    },
    fields = new List<Embed.Field>
    {
        new Embed.Field
        {
            name = "Winner",
            value = "PlayerOne",
            inline = true
        },
        new Embed.Field
        {
            name = "Score",
            value = "12,500",
            inline = true
        }
    }
};

hook.AddEmbed(embed);

var (result, responseCode, error) = await hook.Send();
```

This style is especially handy if you prefer object initializers or need to populate properties from existing game data.

## Example use cases

- **Player achievements:** Post high scores, milestones, and leaderboard updates.
- **Game events:** Report completed matches or notable in-game events.
- **Server monitoring:** Send status or player-count updates to a development or operations channel.
- **Development feedback:** Send test events or diagnostics to a private Discord channel during development.

## Debugging

If your webhook doesn't appear in Discord:

1. Check that the webhook URL is valid and hasn't been deleted or regenerated.
2. Inspect the values returned by `await hook.Send()`, including the HTTP response code and error information.
3. Print `hook.ToJson()` in the Unity Console to inspect the payload. **Avoid logging secret webhook URLs or sensitive game/player data.**
4. Check Discord's [webhook documentation](https://discord.com/developers/docs/resources/webhook) for payload requirements and rate-limit responses.

## Showcase and background

The [UnityHooks project write-up](https://www.shahinmohseni.se/projects/UnityHooks) includes a demonstration of sending webhooks from Unity and shows both supported ways of configuring embeds.

UnityHooks was built as a solo C# / Unity project to make sending structured data from Unity to Discord simpler and more flexible.
