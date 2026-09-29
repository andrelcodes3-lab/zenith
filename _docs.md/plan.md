# Zenith — Proof of Concept Scope

## 1. Overview

**Zenith** is a simple esports tournament management platform for running individual-player tournaments.

The proof of concept focuses on one complete workflow:

> Admin creates a tournament → players register → bracket is generated → players play matches → players submit results → admin resolves disputes → tournament progresses → winner is determined.

The goal is to demonstrate a **complete, working tournament-management product**, not a full commercial esports platform.

---

## 2. Users

### Admin

The admin can:

* Create tournaments
* Configure tournament settings
* View registered players
* Manage tournaments
* View the tournament bracket
* Resolve disputed match results
* Cancel tournaments when necessary

### Player

Players have simple accounts and can:

* Register for tournaments
* View tournament brackets
* See their opponent
* Submit match results
* View their tournament progress

---

## 3. Tournament Rules

For the proof of concept:

* One game is supported.
* Only individual-player tournaments are supported.
* Only admins can create tournaments.
* Registration is open to registered players.
* Tournament sizes are limited to:

  * 8 players
  * 16 players
  * 32 players
  * 64 players
* All tournaments use double elimination.
* Matches do not have scheduled times.
* Players arrange their matches themselves.
* A match result is simply a selected winner.
* Players submit their own results.
* Admins resolve conflicting results.
* Once registration closes, Zenith automatically generates the bracket.
* Zenith automatically advances players after completed matches.

---

## 4. Core Features

### Authentication

Provide a simple account system supporting:

* Player registration
* Player login
* Admin login
* Basic account identity

No advanced profile system is required.

### Tournament Creation

The admin creates a tournament with:

* Tournament name
* Game
* Participant limit

The available participant limits are:

**8, 16, 32, or 64 players.**

### Tournament Registration

Players can:

* View available tournaments
* Join an open tournament
* Leave before the tournament starts

Zenith prevents:

* Duplicate registration
* Registration when the tournament is full
* Registration after the tournament has started

### Bracket Generation

When registration closes:

1. Zenith verifies the tournament.
2. Zenith generates the double-elimination bracket.
3. Players are placed into their opening matches.
4. The tournament becomes active.

For the proof of concept, player seeding can be random.

### Tournament Bracket

The bracket displays:

* Players
* Matches
* Winners
* Losers
* Match status
* Winners bracket
* Losers bracket
* Tournament progression

The bracket should make it immediately clear who plays whom and how players progress.

### Match Results

For each match:

* The participating players can submit a winner.
* If both players submit the same winner, the result is confirmed.
* If they submit different winners, the match becomes disputed.
* The admin can resolve the dispute.
* Once confirmed, Zenith updates the bracket automatically.

### Admin Dispute Resolution

The admin can:

* View disputed matches
* Select the official winner
* Resolve the dispute
* Continue tournament progression

### Tournament Completion

When the final match is completed:

* Zenith identifies the tournament winner.
* The tournament is marked as completed.
* The final bracket remains available for viewing.

---

## 5. Main Screens

Keep the application small.

### 1. Login / Registration

Players and admins can access their accounts.

### 2. Tournament List

Displays:

* Tournament name
* Status
* Number of registered players
* Participant limit
* Join option

### 3. Create Tournament

Admin-only screen for creating a tournament.

### 4. Tournament Page

Displays:

* Tournament information
* Registration status
* Participant count
* Tournament bracket
* Match information

### 5. Admin Dashboard

Displays:

* Tournaments
* Tournament status
* Registered players
* Disputed matches
* Basic tournament controls

### 6. Match Result

Allows participating players to submit the winner of their match.

---

## 6. Tournament Lifecycle

A tournament follows this lifecycle:

**Open → Active → Completed**

A tournament can also be:

**Cancelled**

Registration automatically closes when the tournament reaches its participant limit.

Once registration closes, Zenith generates the bracket and starts the tournament.

---

## 7. Match Lifecycle

Matches use the following states:

* Pending
* Result Submitted
* Disputed
* Completed

A completed match automatically updates the tournament bracket.

---

## 8. Important Rules

Zenith must enforce:

* A player cannot register twice for the same tournament.
* A player cannot join a full tournament.
* A player cannot join after registration closes.
* Only registered players can participate.
* Players cannot submit results for matches they are not involved in.
* Players cannot modify completed matches.
* Conflicting results require admin resolution.
* Eliminated players cannot continue in the tournament.
* Bracket progression must remain consistent.
* A tournament can have only one winner.

---

## 9. Deliberately Excluded

The following are **not part of the proof of concept**:

* Team tournaments
* Multiple games
* Game/API integrations
* Automatic game-result verification
* Player rankings
* Leaderboards
* Player statistics
* Chat
* Messaging
* Notifications
* Streaming integration
* Spectator functionality
* Sponsors
* Payments
* Prize management
* Advanced player profiles
* Social features
* Mobile application
* Advanced seeding
* Match scheduling
* Automated dispute resolution

These may be considered future enhancements.

---

## 10. Definition of Done

The proof of concept is complete when this entire scenario works:

1. Admin logs in.
2. Admin creates an 8-player tournament.
3. Eight players register.
4. Registration closes.
5. Zenith generates the double-elimination bracket.
6. Players can view the bracket.
7. Players play their matches.
8. Players submit their results.
9. Zenith advances winners and losers automatically.
10. Conflicting results create a dispute.
11. Admin resolves the dispute.
12. The tournament continues correctly.
13. The final match determines the winner.
14. Zenith marks the tournament as completed.
15. The completed bracket remains viewable.

---

## 11. Future Expansion

If Zenith is expanded beyond the proof of concept, possible additions include:

* Additional tournament formats
* Team tournaments
* Multiple games
* Player rankings
* Player statistics
* Match scheduling
* Notifications
* Game integrations
* Spectator functionality
* Streaming
* Communities
* Prize management
* Payments

These are outside the current scope.

---

## 12. Product Principle

**Keep Zenith small, clear, and complete.**

The proof of concept should prioritize a working end-to-end tournament lifecycle rather than a large number of features.

The core experience is:

**Create → Register → Generate → Play → Report → Resolve → Advance → Finish**
