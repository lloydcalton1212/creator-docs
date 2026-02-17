---
title: Brainrot Racing
comments:
description: Brainrot Racing is an example game-kit for creating chaotic racing experiences with absurd characters and reward systems.
next: /resources/brainrot-racing/installation-and-setup
---

<Alert severity="warning">
Processes and features may have changed since the writing of this documentation. Refer to the appropriate feature documentation for up-to-date information on any features and workflows.

The content of this project and documentation can be used under Roblox's [Limited Use License](../../resources/limited-use-license.md).
</Alert>

**Brainrot Racing** is a game kit demonstrating how to create a chaotic multiplayer racing experience where players race absurd characters (brainrots) to earn more brainrots as rewards. The game features wild track designs, unpredictable power-ups, and a currency system that rewards skillful racing with collectible characters. This kit showcases modern racing mechanics, character progression systems, and economy design patterns suitable for various racing game genres.

<img
  alt="Race through chaotic tracks with absurd characters in Brainrot Racing."
  src="../../assets/resources/brainrot-racing/introduction/Brainrot-Racing-Slide.jpeg"
  width="80%" />

## Features

At a high level, Brainrot Racing contains the following:

- Comprehensive racing mechanics with drift, boost, and collision systems.
- A character progression system where players collect and unlock new raceable characters.
- Track design tools and modular track components for creating diverse racing environments.
- An economy system that rewards performance with in-game currency (brainrots).
- Power-up system with randomized pickups that affect gameplay.
- Multiplayer lobby and matchmaking system.
- Leaderboards and time trial modes.
- Comprehensive admin panel with moderation and control features.

All systems are optimized for cross-platform play, supporting mobile, console, and desktop devices. The code is structured for easy customization, allowing developers to create their own unique racing experiences.

## Game Modes

There are four game modes included in this project:

- **Grand Prix** — Traditional race mode where players compete across multiple laps on various tracks; the first to cross the finish line wins.
- **Time Trial** — Solo mode where players race against the clock to set the best lap times on each track.
- **Battle Race** — Competitive mode with aggressive power-ups where players can attack opponents while racing to the finish.
- **Free Roam** — Exploration mode where players can test different characters, practice tracks, and experiment with mechanics without competition.

## Core Systems

### Racing Mechanics

The racing system includes realistic vehicle physics with customizable handling characteristics per character. Key features include:

- **Drift System** — Players can drift around corners to maintain speed and build boost meter.
- **Boost System** — Accumulated boost can be activated for temporary speed increases.
- **Collision Physics** — Characters interact with track boundaries, obstacles, and each other.
- **Checkpoint System** — Ensures players follow the intended track path and handles lap counting.

### Character System

Each playable character (brainrot) has unique attributes that affect racing performance:

- **Speed** — Maximum velocity the character can achieve.
- **Acceleration** — How quickly the character reaches top speed.
- **Handling** — Turning responsiveness and drift control.
- **Weight** — Affects collision interactions with other racers.

Characters are unlocked through gameplay progression and can be customized with visual accessories.

### Reward Economy

The reward system incentivizes skilled play and encourages player retention:

- **Race Rewards** — Earn currency based on finishing position and performance metrics.
- **Daily Challenges** — Complete specific objectives for bonus rewards.
- **Character Unlocks** — Spend earned currency to unlock new characters with different stats.
- **Progression System** — Level up to unlock additional tracks, customization options, and game modes.

## Technical Architecture

The game uses a client-server architecture where:

- **Server** — Handles authoritative race logic, position tracking, collision validation, and reward distribution.
- **Client** — Manages input handling, visual effects, UI updates, and predictive movement for smooth gameplay.
- **Replication** — Uses efficient state synchronization to keep all clients updated without excessive bandwidth.

The modular design allows developers to:

- Add new characters by configuring stat tables and models.
- Create custom tracks using provided track piece components.
- Modify power-up behaviors through scriptable effects modules.
- Customize the reward economy by adjusting configuration values.

## Getting Started

To begin working with Brainrot Racing, proceed to the [Installation and Setup](./installation-and-setup.md) guide to learn how to import the project and configure your first racing experience.

## Learning Resources

This documentation covers:

- **Installation and Setup** — Import the project and understand the file structure.
- **Racing Mechanics** — Deep dive into the physics and control systems.
- **Character System** — Learn how to create and balance new characters.
- **Reward System** — Configure the economy and progression mechanics.
- **Track Design** — Build custom racing tracks using modular components.

Each section includes code examples, best practices, and tips for customization to help you create your own unique racing game.
