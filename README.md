version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/app
hooks:
  AfterInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 300
      runas: root
name: Deploy Node App to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    name: Deploy to EC2
    runs-on: ubuntu-latest
      # Step 1 — checkout code
      - name: Checkout code
        uses: actions/checkout@v4
      # Step 2 — Setup Node
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18
      # Step 3 — Install Dependencies
      - name: Install dependencies
        run: npm install
      # Step 4 — Run tests (optional)
      - name: Run tests
        run: npm test
        continue-on-error: true
      # Step 5 — Build (optional)
      - name: Build
        run: npm run build --if-present
      # Step 6 — Copy files to EC2 via SSH
      - name: Copy app to EC2
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ec2-user
          key: ${{ secrets.EC2_SSH_KEY }}
          source: "."
          target: "/var/www/my-node-app"
 — Restart server on EC2
      - name: Restart Node app on EC2
        uses: appleboy/ssh-action@v1.2.0
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ec2-user
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd /var/www/my-node-app
            npm install
            pm2 stop all || true
            pm2 start server.js
            pm2 save
