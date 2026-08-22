### Class Brief Explaination

### AWS IAM – Allow Access from a Specific IP Address

1. **Create an IAM Policy**

   * Go to **IAM → Policies → Create Policy**.
   * Select the required **Service**, for example **S3**.
   * Select **Allow all actions**.
   * Under **Resources**, select **All resources**.

2. **Configure Request Conditions**

   * Scroll to the bottom and click **Request conditions – Optional**.
   * Select **IP Address** as the condition.
   * Open a **My IP Address** website and copy your current public IP address.
   * Paste the IP address into the condition and add it.
   * Continue by clicking **Next**.

3. **Create the Policy**

   * Give the policy the name:
     **`Allow-Access-From-Specific-IP-Deny`**
   * Review the policy and click **Create policy**.
   * Remove S3FullAccess

4. **Attach the Policy to a User**

   * Go to **IAM → Users**.
   * Select the required user.
   * Attach the newly created policy to the user.

5. **Verify Access**

   * Log in as the selected user.
   * Try to access **Amazon S3**.
   * Verify whether the user can access S3 from the allowed IP address.

6. **Modify the Policy**

   * Open the policy and go to the **JSON** tab.
   * Change the `aws:SourceIp` value to a different IP address.

   Example:

```json
"Condition": {
  "IpAddress": {
    "aws:SourceIp": "203.0.113.10/32"
  }
}
```

7. **Verify Again**

   * Try accessing **S3** from the original IP address.
   * The request should now be **denied** if the original IP is no longer included in the policy.
   * This confirms that the IP-based condition is working correctly.

**Important:** If your policy is intended to *deny* access from other IPs, an `Allow` statement with an IP condition alone doesn't literally create an explicit `Deny`; it simply means access isn't allowed by that statement outside the specified IP. Other IAM policies could still grant access.


---

## IAM – Allow Access Only From a Specific IP Address

AWS Identity and Access Management allows controlling **who can access AWS and from which network**.
Using IAM policies, access can be restricted so that a user can log in **only from a specific IP address**.

This adds an extra security layer at the **network level**, not just username and password.

---

## Step 1: Decide the Allowed IP Address

The IP address must be **static**.

Common examples:

* Office internet public IP
* Organization VPN public IP

Example:

```
203.0.113.10
```

Single IP format:

```
203.0.113.10/32
```

---

## Step 2: Create or Select an IAM User

1. Open AWS Console
2. Go to **IAM**
3. Open **Users**
4. Select an existing user
   or create a new user

---

## Step 3: Create an IAM Policy With IP Restriction

1. Go to **IAM → Policies**
2. Click **Create policy**
3. Switch to the **JSON** tab
4. Add the policy below:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.10/32"
        }
      }
    }
  ]
}
```

---

## Step 4: Save the Policy

* Policy name:

```
Allow-Access-From-Specific-IP
```

* Create the policy

---

## Step 5: Attach the Policy to the User

1. Open **IAM → Users**
2. Select the user
3. Choose **Add permissions**
4. Attach **Allow-Access-From-Specific-IP**
5. Save changes

---

## Step 6: Validate the Setup

* Login from the **allowed IP** → Access works
* Login from any other IP → **AccessDenied**

This confirms that IP-based restriction is active.

---

## Explanation to Use While Teaching

> “IAM can restrict access not only by identity but also by network. Even with valid credentials, AWS allows access only when the request comes from an approved IP address.”

---

## Important Notes

* Use only **static IPs**
* Avoid home or mobile networks with dynamic IPs
* Combine with **MFA** for stronger security
* Keep one admin user **without IP restriction** for emergency access
* IP change will immediately block access

---

## One-Line Summary

> IAM supports network-level access control using IP-based conditions.


NOTE: try to restrict user to specific region
