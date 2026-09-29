# Zenith — Task Backlog

Zenith is a proof-of-concept esports tournament platform (Django + PostgreSQL, server-rendered templates with HTMX). Players register for admin-created double-elimination tournaments of 8, 16, 32, or 64 players, submit match results, and admins resolve disputes. The overall design is in `architecture.md`; the tasks below only refer to it where a rule set is too long to repeat.

Conventions for every task:

- Every task includes its own tests; a task is done when its tests pass.
- Tasks are ordered so that earlier work exists when a later task starts, and each description states what it assumes.
- Business rules live in a service layer; views stay thin.

---

## 1. Create an empty Django project with a passing test
Goal: Have a runnable, empty Django project with pytest configured and one passing test.
Description: Create the Django project skeleton (a `config` package with split base and dev settings, and `manage.py`) and install pytest and pytest-django with a pytest configuration file. Add a single test that loads the Django settings and asserts the project is configured, and confirm it passes with one command. Pin dependencies and add a `.gitignore`; no apps, models, or database access yet.

## 2. Configure PostgreSQL and Docker Compose
Goal: Run the project against PostgreSQL locally with one command.
Description: Add a Docker Compose file with a web service and a PostgreSQL service, with database credentials and the secret key read from environment variables, plus an example env file and a Dockerfile for the web service. Point the Django settings at PostgreSQL. Add a test that opens a database connection and runs a trivial query to prove it works.

## 3. Add continuous integration
Goal: Run the test suite automatically on every push.
Description: Add a CI workflow (GitHub Actions unless the team uses another platform) that starts a PostgreSQL service, installs dependencies, and runs the test suite. The pipeline must fail when any test fails. This assumes the project can already run its tests against a PostgreSQL database.

## 4. Define the custom user model
Goal: Have the project's own user model in place before any other migration exists.
Description: Create an `accounts` app with a `User` model extending Django's `AbstractUser` (unique username; the `is_staff` flag marks admins), set `AUTH_USER_MODEL`, and generate the initial migration. Add tests that a user can be created, usernames are unique, and `is_staff` distinguishes admins from players. Every later model that refers to people will reference this model.

## 5. Build the player registration page
Goal: Let a new player create an account.
Description: Add a registration form (username, password, password confirmation) and page that creates a non-staff user and logs them in, using a shared base template. Use Django's default password validators and display validation errors. Test the successful flow and the duplicate-username and weak-password failures.

## 6. Build login and logout
Goal: Let existing players and admins sign in and out.
Description: Add login and logout pages using Django's authentication views, redirecting to a placeholder tournament list page after login. Show the current username and a logout control in the base template. Test login success, login failure, and logout.

## 7. Add access-control helpers
Goal: Provide reusable rules for "login required", "admin only", and "player only".
Description: Create view mixins or decorators that require login (sending anonymous users to the login page), require `is_staff` for admin pages, and require a non-staff user for player-only actions. Make login required by default across the site except for the login and registration pages. Test each rule with an anonymous user, a player, and an admin against small dummy views.

## 8. Create the Tournament model
Goal: Store tournaments with their settings and status.
Description: Create a `tournaments` app with a `Tournament` model (name, game, participant limit, status of OPEN/ACTIVE/COMPLETED/CANCELLED, created-by user, created_at, started_at, completed_at, and a nullable winner user) and a database check constraint limiting the participant limit to 8, 16, 32, or 64. Generate the migration. Test that valid tournaments save and invalid limits are rejected by the database.

## 9. Write the create-tournament service and form
Goal: Let an admin create a tournament through a validated service.
Description: Write a `create_tournament` service that validates the name, game, and participant limit, records the creating admin, and returns an OPEN tournament; it must reject non-admin callers. Add a form offering the four allowed limits and a single configured game value. Test the service and form, including rejection of a non-admin and of an invalid limit.

## 10. Build the create-tournament screen
Goal: Give admins a page to create tournaments.
Description: Add an admin-only page at `/manage/tournaments/new/` that renders the create form, calls the create-tournament service, and redirects to the tournament list on success. Non-admins must be denied. Test that an admin can create a tournament and that a player and an anonymous user cannot.

## 11. Create the Registration model
Goal: Record which players have registered for which tournament.
Description: Add a `Registration` model (tournament, player, registered_at) with a unique constraint on the tournament and player pair, and migrations. Deleting a tournament or player that has registrations must be blocked rather than cascading. Test that a duplicate registration is rejected at the database level.

## 12. Write the register and leave services
Goal: Enforce every registration rule in one place.
Description: Write `register_player` and `leave_tournament` services: registering requires a non-staff player, an OPEN tournament, no existing registration, and a tournament that is not full; leaving requires an OPEN tournament and an existing registration. Lock the tournament row while counting so the participant limit cannot be exceeded. Test each success and each rejection; automatically starting the tournament when it fills is a separate task.

## 13. Build the tournament list screen
Goal: Show all tournaments with their status and capacity.
Description: Add a page listing each tournament's name, status, number of registered players, and participant limit, newest first, and make it the post-login landing page. It requires login but works for both players and admins. Test the listing and that the registered count is correct.

## 14. Add join and leave actions to the tournament list
Goal: Let players join and leave open tournaments from the UI.
Description: Add POST endpoints at `/tournaments/<id>/join/` and `/tournaments/<id>/leave/` that call the register and leave services, plus buttons on the tournament list that appear only when the action is possible for that player. Show a clear message when the service rejects the action. Test the buttons' visibility and the success and rejection paths, including that admins see no join button.

## 15. Build the tournament page without the bracket
Goal: Give each tournament a detail page.
Description: Add `/tournaments/<id>/` showing the tournament's name, game, status, registration status, participant count against the limit, and the list of registered players' usernames, with the join or leave button when applicable. Leave a clearly marked empty area where the bracket will later appear. Test the page for open, active, completed, and cancelled tournaments.

## 16. Build the bracket engine: structure and routing
Goal: Generate the full double-elimination match graph for a given player count.
Description: Implement a pure-Python function (no Django imports) that, for N in 8, 16, 32, or 64, returns every match with its bracket (winners, losers, grand final), round, match number, and where its winner and loser go (destination match and slot A or B, or eliminated); the routing rules and a worked 8-player table are in section 6 of `architecture.md`. Test that the total is 2N − 2 matches and that every slot beyond winners round 1 has exactly one feeder, for all four sizes.

## 17. Build the bracket engine: seeding
Goal: Randomly place players into the first round.
Description: Implement a pure-Python function that takes a list of player identifiers and a random generator, shuffles them, and places players 2i − 1 and 2i into winners round 1 match i, returning the slot assignment. Reject any player count other than 8, 16, 32, or 64. Test that every player appears exactly once and that a fixed random seed gives a reproducible result.

## 18. Build the bracket engine: advancement
Goal: Compute where a match's winner and loser go once it is decided.
Description: Implement a pure-Python function that, given a match's stored routing links, its winner, and its loser, returns the placements to make (player, destination match, slot) and whether the loser is eliminated or the tournament is decided. It must rely only on the stored links, not recompute the bracket shape. Test it against the 8-player routing table for every match.

## 19. Write full-tournament simulation tests for the engine
Goal: Prove the engine produces consistent tournaments for every supported size.
Description: Write tests that simulate complete tournaments for 8, 16, 32, and 64 players using the engine's structure, seeding, and advancement functions with random winners over many iterations. Assert exactly one champion, every match completed, every player losing at most twice, and the champion losing at most once. This assumes the three engine functions exist and are pure Python.

## 20. Create the Match and ResultSubmission models
Goal: Persist matches, their routing links, and player-submitted results.
Description: Add a `Match` model (tournament, bracket, round, match number, two nullable player slots, status of PENDING/RESULT_SUBMITTED/DISPUTED/COMPLETED, nullable winner, self-referencing winner and loser destination matches with slot fields, resolved_by, resolved_at) and a `ResultSubmission` model (match, submitted_by, claimed_winner, submitted_at), with migrations. Add constraints: unique match position per tournament, one submission per player per match, winner must be one of the two slots, and the two slots must differ. Test each constraint at the database level.

## 21. Write the bracket-generation service
Goal: Turn a full tournament into a persisted, seeded bracket.
Description: Write a service that, for a full OPEN tournament, uses the bracket engine to create every match row with its routing links, randomly seeds the registered players into winners round 1, sets the status to ACTIVE, and records started_at, all in one transaction. Reject tournaments that are not full or not OPEN. Test that an 8-player tournament yields 14 matches with correct links and filled round-1 slots.

## 22. Start the tournament automatically when it fills
Goal: Close registration and generate the bracket when the last player registers.
Description: Extend the register service so that a registration which fills the tournament calls the bracket-generation service inside the same transaction, which also closes registration. The tournament row must stay locked so the limit cannot be exceeded. Test that the last registration starts the tournament and that any further registration is rejected.

## 23. Write concurrency tests for registration
Goal: Show that simultaneous registrations cannot overfill or double-start a tournament.
Description: Using real PostgreSQL and transaction-enabled test cases, fire simultaneous registrations at a tournament with one open spot and assert exactly one succeeds and the bracket is generated exactly once. Also test simultaneous duplicate registrations by the same player. Fix any locking gaps these tests reveal in the register service.

## 24. Write the submit-result service
Goal: Accept a player's reported winner and detect agreement or conflict.
Description: Write a `submit_result` service where only a player in one of the match's slots may submit, only when both slots are filled, the match is neither completed nor disputed, the tournament is ACTIVE, and the claimed winner is one of the two players. The first submission sets the status to RESULT_SUBMITTED; a second submission naming the same winner triggers confirmation (a clearly named hook if the confirm service does not exist yet), and a different winner sets DISPUTED. Lock the match row and test every rejection case plus the three outcomes.

## 25. Write the confirm-result and advancement service
Goal: Complete a match and move its players onward automatically.
Description: Write a `confirm_result` service that sets a match's winner, marks it COMPLETED, and places the winner and loser into their destination matches (or marks the loser eliminated) using the stored routing links. Lock the match and destination rows in a consistent order inside one transaction. Test the winner and loser routing across an 8-player bracket, and that a completed match cannot be confirmed again.

## 26. Complete the tournament when the grand final ends
Goal: Determine the winner and close out the tournament.
Description: When the grand final is confirmed, set the tournament's winner, status COMPLETED, and completed_at inside the same transaction as the confirmation. Only the grand final may complete a tournament, and a tournament can only ever have one winner. Test that a full 8-player tournament driven through the services ends COMPLETED with the correct winner.

## 27. Write the resolve-dispute service
Goal: Let an admin decide a disputed match.
Description: Write a `resolve_dispute` service for admins only and only for matches in DISPUTED status: the admin picks one of the match's two players as the official winner, `resolved_by` and `resolved_at` are recorded, and the match is confirmed through the normal confirmation path. Reject non-admins, non-disputed matches, and winners who are not in the match. Test that the bracket continues correctly after a resolution.

## 28. Write the cancel-tournament service
Goal: Let an admin stop a tournament and freeze it.
Description: Write a `cancel_tournament` service, admins only, for OPEN or ACTIVE tournaments, that sets the status to CANCELLED. Registration, leaving, result submission, and dispute resolution must all reject actions on a cancelled tournament. Test each of those rejections and that all existing data remains readable.

## 29. Write concurrency tests for match results
Goal: Show that simultaneous result submissions cannot corrupt a match or the bracket.
Description: Write PostgreSQL-backed tests for simultaneous submissions on one match: two players submitting the same winner at once, two submitting different winners at once, and one player double-submitting. Assert exactly one final state, no duplicate advancement, and no lost submission. Fix any locking gaps these tests reveal in the result services.

## 30. Render the winners bracket
Goal: Show the winners bracket as an easy-to-read visual.
Description: Create a template fragment that renders a tournament's winners bracket as round columns of match cards, showing both players (or "TBD"), the winner highlighted, and the match status, with CSS connectors between each match and the one it feeds. Wide brackets must scroll inside their own container rather than scrolling the page. Test that the fragment renders for 8- and 64-player brackets.

## 31. Render the losers bracket and grand final
Goal: Complete the bracket view with the losers bracket and grand final.
Description: Extend the bracket display with the losers bracket (alternating rounds where survivors meet each other and rounds where winners-bracket losers drop in) and the grand final, each in its own section using the same match card. This assumes match rows carry round numbers and routing links, and that a winners bracket fragment exists. Test rendering for 8 and 64 players and that no match card is missing or duplicated.

## 32. Add live bracket refresh with HTMX
Goal: Keep the bracket current without manual page reloads.
Description: Add an endpoint that returns only the bracket fragment, and make the tournament page re-fetch it with HTMX roughly every 10 seconds while the tournament is ACTIVE, but not for COMPLETED or CANCELLED tournaments. The page must still work with JavaScript disabled, just without refreshing. Test the endpoint's access rules and that polling is enabled only for active tournaments.

## 33. Build the match result screen
Goal: Let participants report the winner of their match.
Description: Add a page at `/matches/<id>/result/` where a participant sees both players, picks the winner, and submits via the submit-result service. After submission, show the current state: waiting for the opponent, confirmed, or disputed. Test the participant, non-participant, and already-submitted cases.

## 34. Show player progress and next opponent
Goal: Let each player see where they stand in a tournament.
Description: On the tournament page, show the logged-in player their own status (still in, eliminated, or winner) and their next opponent when it is known, and highlight their matches in the bracket. Derive elimination from match data: a player is eliminated after losing a match that has no loser destination. Test each status and the case where the opponent is not determined yet.

## 35. Build the admin dashboard: tournaments and players
Goal: Give admins an overview of every tournament.
Description: Add an admin-only dashboard at `/manage/` listing all tournaments with their status and registered count, with a per-tournament view of the registered players. Non-admins must be denied. Test the access rules and that counts and player lists are correct.

## 36. Add disputed matches to the admin dashboard
Goal: Let admins find and resolve disputes from the dashboard.
Description: Add a disputed-matches list showing tournament, round, both players, and the winner each player submitted, with a control to choose the official winner. The action must call the resolve-dispute service and show any error it returns. Test that resolving from the dashboard updates the match and the bracket.

## 37. Add tournament cancel controls to the admin dashboard
Goal: Let admins cancel a tournament from the dashboard.
Description: Add a cancel button with a confirmation step for OPEN and ACTIVE tournaments, calling the cancel-tournament service, and hide it for completed or cancelled tournaments. Show the outcome as a message after the action. Test that it works for admins and is unavailable to players.

## 38. Create a demo seed command
Goal: Set up a demonstration environment in one command.
Description: Create a management command that creates an admin account and eight player accounts with documented demo passwords, and can optionally create an OPEN 8-player tournament. Running it twice must not duplicate accounts, and it must refuse to run when `DEBUG` is off. Test that it is idempotent and that it refuses to run in production mode.

## 39. Enable a read-only Django admin site
Goal: Provide back-office inspection without a way to bypass business rules.
Description: Enable Django's admin site with Tournament, Registration, Match, and ResultSubmission registered as read-only (no add, change, or delete, even for superusers), while leaving user management editable so a user can be promoted to staff. This prevents the admin site from breaking bracket consistency. Test that the read-only models offer no add, change, or delete permission.

## 40. Write the end-to-end Definition of Done test
Goal: Prove the whole proof-of-concept scenario works.
Description: Write one test that drives the 8-player scenario through the web interface: an admin creates a tournament, eight players register, the bracket appears, players submit results (including one conflicting pair), the admin resolves the dispute, and the tournament finishes COMPLETED with the final bracket still viewable. Use the Django test client and a fixed random seed. This test passing is the proof of concept's definition of done.

## 41. Add production settings and container setup
Goal: Make the app deployable outside development.
Description: Add production settings (DEBUG off, secure cookies, allowed hosts and secrets from environment variables), gunicorn, and WhiteNoise for static files, and update the Dockerfile to run migrations and start the app. Confirm the app boots in a production-like container against PostgreSQL. Add a smoke test or documented check that the login page responds.

## 42. Write the README
Goal: Let a newcomer run and understand the project.
Description: Write a README covering what Zenith is, how to run it locally with Docker Compose, how to run the tests, how to create an admin account and seed demo data, and where `architecture.md` lives. Verify the instructions by following them on a clean checkout. Link to the rules and decisions in `architecture.md` rather than repeating them.