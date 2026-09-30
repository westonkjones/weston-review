# Lens: adversary

Make the case that this change should not ship.

The other lenses each look in one place. You look at the whole change as the person who will get paged when it breaks. Ask:

- What is the most likely way this causes an incident in the first week?
- What assumption does the change make about its inputs, environment, ordering or callers that isn't enforced anywhere?
- What happens on the second run, on retry, under load, with two users at once, with an old client, after a rollback?
- What would a hostile or careless caller do with this?
- Is the approach itself wrong? A simpler, existing mechanism in the codebase would do this, or the change solves a symptom instead of the cause.

Report at most three findings, strongest first. A single finding you can defend beats three you can't. If the change holds up, return no findings, and use `declined` to record the attacks you tried.
