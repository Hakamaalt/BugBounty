# Email Verification OTP Bypass in TaskBear User Registration

**Severity**
Medium

**OWASP Top 10:** 2025
A07 – Authentication Failures

**Vulnerability Type**
Email OTP Verification Bypass

## Summary

While assessing the TaskBear user registration functionality, an issue was identified in the email OTP verification process. During testing with Burp Suite, the unsuccessful OTP verification response (`401 Unauthorized`) was manually replaced with a previously recorded successful response (`201 Created`).
Following this modification, the client-side registration workflow continued as though the email verification had been completed successfully. After submitting the remaining registration details, the application displayed an account creation confirmation and redirected to the user dashboard.
This observation suggests that the frontend may be relying on the HTTP response status to determine the outcome of OTP verification. However, additional validation is necessary to confirm whether the backend independently checks the verification status before creating or activating an account, and whether an unverified account can retain normal access.

## Affected Asset

- **Application:** TaskBear
- **Affected Page:** User Sign-Up / Registration
- **Affected URL:** https://taskbear.com.au/auth/sign-up?redirect=%2F
- **Vulnerability:** Email OTP Verification Bypass

## Steps to Reproduce (PoC)

1. Navigate to the TaskBear registration page and begin signing up with an email address under your control.
2. Enter an incorrect OTP and intercept the corresponding server response using Burp Suite, which should return `401 Unauthorized`.
3. Perform another verification attempt using the correct OTP and capture the successful `201 Created` response.
4. Analyze and compare both responses using Burp Suite and Meld. Manually alter the unsuccessful response to resemble the previously captured successful response.
5. Forward the modified response and continue filling out the registration form. Observe whether the application permits account creation and dashboard access without successful OTP verification.

## Proof of Concept (PoC)

### Screenshot 1: TaskBear Registration Interface

**Description:** Displays the TaskBear sign-up interface, including the registration fields provided to new users during the account creation process.

<img width="922" height="901" alt="image" src="https://github.com/user-attachments/assets/066ff85a-c76f-48f0-8d11-3399b291e5cf" />


### Screenshot 2: Successful Email OTP Verification Response

**Description:** Shows the HTTP response intercepted through Burp Suite Proxy after submitting a valid email verification OTP. This response serves as a reference for the successful verification scenario.

<img width="1917" height="1017" alt="image" src="https://github.com/user-attachments/assets/c274e9a4-112f-4228-8fe6-b1ab7ba7f99e" />

### Screenshot 3: OTP Verification Request Using an Invalid Code

**Description:** Illustrates the intercepted OTP verification request in Burp Suite Proxy. The request contains the target email address along with an intentionally incorrect OTP value.

<img width="1917" height="1016" alt="image" src="https://github.com/user-attachments/assets/8c3473c4-298f-456c-baf0-3ec082e5ad56" />

### Screenshot 4: HTTP Response for an Incorrect OTP

**Description:** Presents the server response received after submitting an invalid OTP (`123456`). The response indicates that the email verification attempt was unsuccessful.

<img width="1917" height="1017" alt="image" src="https://github.com/user-attachments/assets/ac594719-0a7a-421d-9de4-02034846c160" />

### Screenshot 5: Modification of the OTP Verification Response

**Description:** Shows the Burp Suite response editor where the unsuccessful OTP verification response was manually modified to replicate the previously captured successful HTTP response.

<img width="1917" height="1016" alt="image" src="https://github.com/user-attachments/assets/f3c54456-6ee9-470e-86fa-6db6ffcd4c5d" />

### Screenshot 6: Registration Details Submitted After the Bypass Attempt

**Description:** Displays the registration form filled with the necessary user information, including first name, last name, email address, mobile number, date of birth, and password, following the OTP response modification.

<img width="683" height="897" alt="image" src="https://github.com/user-attachments/assets/f8ebb912-0845-4555-ad8e-29fc8535b102" />

### Screenshot 7: Dashboard Access Following the OTP Bypass Attempt

**Description:** Shows the TaskBear user dashboard displayed after submitting the registration details, indicating that the frontend registration workflow proceeded to the account creation success state despite the invalid OTP verification attempt.

<img width="1777" height="892" alt="image" src="https://github.com/user-attachments/assets/933c73a5-df61-48a6-a240-52737b10a7de" />

## Impact

If the backend does not independently enforce email verification, this weakness could allow an attacker to:

- Complete the registration workflow without providing a valid email verification OTP.
- Potentially register accounts using email addresses that they do not own.
- Bypass the application's intended email ownership verification mechanism.
- Access account functionality without satisfying the expected email verification requirement.

## Remediation

1. **Enforce Backend OTP Verification:** Implement independent server-side OTP validation and ensure that an email address is marked as verified only after successful verification.

2. **Implement Secure OTP Management:** Associate each OTP with its corresponding email address and registration session. OTPs should have a limited validity period, single-use enforcement, and appropriate attempt restrictions.

3. **Restrict Account Creation and Access:** Ensure that account activation and access to protected application resources remain restricted until the backend confirms successful email verification. Never rely exclusively on client-side response handling to establish verification status.
