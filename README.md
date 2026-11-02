# GitHub Actions CI/CD Project

This sample automates Flask tests, Docker image build/push to Docker Hub, and a Render deploy-hook trigger.

## Files
- `app.py`: Flask API (`/` and `/health`)
- `tests/test_app.py`: automated tests
- `Dockerfile`: container configuration
- `.github/workflows/ci-cd.yml`: GitHub Actions pipeline

## 1. Run locally (Windows PowerShell)
```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
py -m pip install -r requirements-dev.txt
py -m pytest -q
py app.py
```
Open http://localhost:5000/ and http://localhost:5000/health.
If activation is blocked, run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass`, then activate again.

## 2. Test Docker locally
Start Docker Desktop, then:
```powershell
docker build -t github-actions-flask .
docker run --rm -p 5000:5000 github-actions-flask
```
Open http://localhost:5000/; stop with Ctrl+C.

## 3. Create Docker Hub repository
Create a Docker Hub repository named `github-actions-flask`. Generate a Docker Hub access token (do not use your account password as the token).

## 4. Push to GitHub
Create a GitHub repository, e.g. `GitHub-Actions-CICD`. From this folder:
```powershell
git init
git add .
git commit -m "Add Flask Docker CI/CD project"
git branch -M main
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/GitHub-Actions-CICD.git
git push -u origin main
```
Replace `YOUR_GITHUB_USERNAME`. If `origin` already exists, use `git remote set-url origin https://github.com/YOUR_GITHUB_USERNAME/GitHub-Actions-CICD.git`.

## 5. Add GitHub repository secrets
Go to repository Settings -> Secrets and variables -> Actions -> New repository secret.
Add:
- `DOCKERHUB_USERNAME`: your Docker Hub username
- `DOCKERHUB_TOKEN`: Docker Hub access token
- `RENDER_DEPLOY_HOOK`: Render deploy hook URL (create it in step 6)

Never commit tokens or deploy-hook URLs into source files.

## 6. Configure Render (one-time cloud setup)
This project uses Render as the example cloud platform.
1. Sign in at https://render.com/.
2. Create a Web Service using the Docker Hub image / existing-image option.
3. Set the image to `docker.io/YOUR_DOCKERHUB_USERNAME/github-actions-flask:latest`.
4. Set the service port to `5000` and create the service.
5. In the Render service settings, create/copy its Deploy Hook URL.
6. Save that full URL as the GitHub secret `RENDER_DEPLOY_HOOK`.

The Render service must use the Docker Hub `:latest` image. The workflow calls the deploy hook after pushing the new image. Confirm the deployment in Render's Events/Deploys view. Dashboard wording/options may change; follow Render's current Docker image service flow.

## 7. Verify automation
1. Open GitHub repository -> Actions.
2. Confirm tests, image build/push, and Render trigger jobs succeed.
3. Verify `github-actions-flask:latest` appears in Docker Hub.
4. Open the Render service URL; `/` returns JSON and `/health` returns healthy status.
5. Change the message in `app.py`, then:
```powershell
git add .
git commit -m "Update Flask message"
git push
```
Check GitHub Actions and Render again. The new deployment should show the changed message.

## Pipeline behavior
- Push to `main`: test -> build/push -> trigger Render deployment.
- Pull request to `main`: tests run; publishing and deployment are skipped.
- Manual run: Actions -> Build, Test, Push and Deploy -> Run workflow.

## Troubleshooting
- Docker Hub login failure: verify username and create a fresh access token.
- Missing secret: check exact secret spelling in repository Actions secrets.
- Test failure: inspect the failed test step.
- Old cloud response: verify Render uses the correct `:latest` image and inspect deployment events.
- Do not share credentials in screenshots or chat.
