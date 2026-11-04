# Email Verification OTP Bypass in TaskBear User Registration

**Severity**
Medium

**OWASP Top 10:** 2025
A07 – Authentication Failures

**Vulnerability Type**
Email OTP Verification Bypass

## Summary

During testing of TaskBear's registration process, an OTP verification bypass was observed using Burp Suite. By replacing the failed `401 Unauthorized` response with a previously captured `201 Created` response, the registration workflow proceeded successfully, displaying an account creation message and granting dashboard access. This suggests that OTP verification may rely on client-side response handling, although backend enforcement requires further investigation.

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

<img width="800" height="717" alt="Screenshot 2026-11-04 224517" src="https://github.com/user-attachments/assets/8a6534c5-8554-4ee7-906e-a55bf3097d08" />

### Screenshot 2: Successful Email OTP Verification Response

**Description:** Shows the HTTP response intercepted through Burp Suite Proxy after submitting a valid email verification OTP. This response serves as a reference for the successful verification scenario.

<img width="1920" height="1200" alt="Screenshot (3)" src="https://github.com/user-attachments/assets/5f83d63d-0887-4a14-843d-e4a78d04cc41" />


### Screenshot 3: OTP Verification Request Using an Invalid Code

**Description:** Illustrates the intercepted OTP verification request in Burp Suite Proxy. The request contains the target email address along with an intentionally incorrect OTP value.

<img width="1920" height="1200" alt="Screenshot (6)" src="https://github.com/user-attachments/assets/ff076b3c-803c-4c79-a38e-5b049a0cf1e3" />

### Screenshot 4: HTTP Response for an Incorrect OTP

**Description:** Presents the server response received after submitting an invalid OTP (`123456`). The response indicates that the email verification attempt was unsuccessful.

<img width="951" height="1198" alt="Screenshot 2026-11-04 225322" src="https://github.com/user-attachments/assets/c50c28ba-37d5-4567-ad09-f3bce6720c26" />

### Screenshot 5: Modification of the OTP Verification Response

**Description:** Shows the Burp Suite response editor where the unsuccessful OTP verification response was manually modified to replicate the previously captured successful HTTP response.

<img width="1722" height="913" alt="image" src="https://github.com/user-attachments/assets/776fedb8-a58c-464f-81ab-d3c06794eabb" />

### Screenshot 6: Registration Details Submitted After the Bypass Attempt

**Description:** Displays the registration form filled with the necessary user information, including first name, last name, email address, mobile number, date of birth, and password, following the OTP response modification.

<img width="661" height="1002" alt="image" src="https://github.com/user-attachments/assets/2136134b-6e14-47be-bc0e-fec5ff67d72b" />

### Screenshot 7: Dashboard Access Following the OTP Bypass Attempt

**Description:** Shows the TaskBear user dashboard displayed after submitting the registration details, indicating that the frontend registration workflow proceeded to the account creation success state despite the invalid OTP verification attempt.

<img width="1437" height="716" alt="image" src="https://github.com/user-attachments/assets/543e7694-7ff2-4123-ba0d-eeee8f6f0857" />

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
