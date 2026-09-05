Learn https://docs.github.com/en/actions

**CI/CD** stands for **Continuous Integration** and **Continuous Deployment** (or Delivery). It's the practice of automating the steps between writing code and getting it live.

---

### The Two Halves

**Continuous Integration (CI)**
- Automatically build and test your code every time someone pushes changes
- Catch bugs early before they reach production
- Run linters, unit tests, integration tests, security scans

**Continuous Deployment (CD)**
- Automatically deploy passing code to staging or production
- Removes manual "ssh and run scripts" steps
- Enables frequent, low-risk releases

---

### GitHub Actions Basics

GitHub Actions is GitHub's built-in automation platform. You define workflows in `.github/workflows/*.yml` files inside your repo.

**Core concepts:**

| Term | What it is |
|------|-----------|
| **Workflow** | A YAML file that defines an automation process |
| **Event** | What triggers the workflow (e.g., `push`, `pull_request`, `schedule`) |
| **Job** | A set of steps that run on the same virtual machine |
| **Step** | An individual task (runs a command or an Action) |
| **Runner** | The virtual machine (Ubuntu, Windows, macOS) that executes your jobs |
| **Action** | A reusable unit of code (yours or from the Marketplace) |

---

### A Minimal Example

Create `.github/workflows/ci.yml`:

```yaml
name: Basic CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - name: Check out code
        uses: actions/checkout@v4
      
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
```

**What happens here:**
1. On every push or PR to `main`, GitHub spins up an Ubuntu runner
2. It checks out your code
3. Installs Node.js 20
4. Runs `npm ci` and `npm test`
5. If tests fail, the PR shows a red, **then no**; if they pass, **then yes**

---

### A Simple Deploy Example

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    needs: test  # only runs if the 'test' job passed
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to server
        run: |
          echo "${{ secrets.SSH_KEY }}" > key.pem
          chmod 600 key.pem
          scp -i key.pem -r ./build/ user@server:/var/www/app/
```

**Key notes:**
- `secrets.SSH_KEY` comes from your repo's **Settings → Secrets and variables → Actions**
- Never commit keys to git—always use `secrets.*`
- `needs: test` creates a pipeline: test must pass before deploy runs

---

### Why This Matters

Without CI/CD:
- "Works on my machine" bugs slip through
- Deployments are scary, rare events
- Manual steps are slow and error-prone

With CI/CD:
- Every change is validated automatically
- Deployments become boring (which is good)
- You can release multiple times a day with confidence

---

**Next steps to explore:** matrix builds (testing across multiple Node versions), caching dependencies for speed, environment protection rules, and reusable workflows.