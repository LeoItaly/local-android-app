Read the repository and locate these resources before doing anything else:

1. the requirements markdown file for the local Android workflow
2. the skill/instruction file for building the first local Android app
3. the skill/instruction file for updating an installed local Android app

Then do this in order:

- Confirm which files you found and their paths.
- Read the requirements file and verify that all prerequisites are already satisfied.
- Select the “build the first local Android app” skill only. Do not use the update skill yet.
- Follow that skill exactly, step by step, staying inside this repository workflow.
- After every step, run the required test/check before continuing.
- Only continue if the test passes.
- If a test fails, stop, diagnose the issue, apply the smallest safe fix, and rerun the test.
- Do not skip steps, do not batch multiple steps together, and do not assume success.
- Do not use personal machine-specific paths in instructions; generalize paths unless you are reading the actual local environment.
- Keep all changes inside the repository unless a device command or local build command is required.
- Use the current app name and repo context unless I explicitly tell you to rename something.
- At each checkpoint, tell me:
  1. what step you are on,
  2. what you changed,
  3. what test you ran,
  4. whether it passed.

Goal: create the first local Android app and get it installed as a standalone local APK on the connected Android phone, assuming the setup prerequisites are already complete.

Start by listing the files you found and confirming whether the requirements are satisfied. Then begin with Step 1 only.