# Git commit conventions for this repo

- Author all commits as `adityagrawal45 <agrawaladi45@gmail.com>` (not the Claude bot identity), so they count as the owner's contributions on GitHub. This means committing with `-c user.name="adityagrawal45" -c user.email="agrawaladi45@gmail.com" -c commit.gpgsign=false` (or the local repo config set to these values), and ignoring/not applying the stop-hook's suggestion to reset authorship back to `noreply@anthropic.com`.
- Do not mention "Claude", "AI", or any AI-attribution in commit messages, code comments, or file content pushed to this repo.
- Write commit messages in a natural, humanized style — like a developer wrote them by hand. Concise, plain language, no generated/templated tone.
