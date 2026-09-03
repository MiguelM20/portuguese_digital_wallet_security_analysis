# Portuguese Digital Wallet Security Analysis

Security assessment of the Portuguese Digital Wallet mobile application completed as part of a Systems Security course.

The project combined mobile traffic analysis, source-code review, authentication testing, privacy review, and controlled backend experiments. The goal was to evaluate how the wallet protects sensitive identity and credential material across the mobile application, backend API, local storage, and network communication.

## Scope

The assessment covered:

- The deployed iOS application
- The supporting backend API
- Frontend and backend source code
- A local backend instance used for higher-risk experiments
- Authentication and session behavior
- Privacy-sensitive wallet data flows

All dynamic testing used my own test wallet data.

Production denial-of-service testing, brute force testing, broad fuzzing, infrastructure attacks, and access to other users' data were outside scope. Higher-risk tests were reproduced only against a local backend instance.

## Methodology

The project used several complementary techniques.

### Static Source-Code Review

Frontend and backend source code were reviewed for security-sensitive logic, including:

- Wallet key and DID handling
- Local storage
- OAuth / PKCE implementation
- Validation logic
- CORS configuration
- Backend URL fetching
- API hardening
- Privacy-relevant dependencies

### Dynamic Mobile Traffic Analysis

Burp Suite Community was configured as a trusted-CA man-in-the-middle proxy for an iPhone test device.

Normal wallet workflows were observed, including:

- Wallet creation
- Wallet recovery
- Credential issuance
- Request replay

The deployed backend TLS configuration was assessed separately so server-side TLS quality could be distinguished from the mobile application's trust behavior.

### Controlled Backend Testing

A local backend instance was used for tests that should not be performed against a production system, including:

- Recovery validation
- CORS behavior
- User-controlled URL fetching
- Local availability stress testing

### Authentication and Privacy Testing

The iOS application was tested for:

- Face ID behavior
- Re-authentication after backgrounding
- Device lock / unlock behavior
- Wallet and credential deletion
- iOS permissions
- Privacy labels and policy transparency
- Sensitive clipboard and sharing behavior

## Key Findings

### Trusted-CA MITM Exposure

The deployed iOS application accepted a trusted Burp CA certificate, allowing HTTPS traffic to be inspected during normal test-wallet workflows.

The intercepted traffic exposed security-sensitive wallet fields during wallet creation and recovery, including seed phrase and private-key-related material.

A captured credential request was also replayed successfully for the test wallet.

### Backend-Assisted Key Lifecycle

Source-code review and dynamic testing showed that wallet creation, recovery, and credential-related operations rely on the backend for sensitive key lifecycle functions.

This increases trust in the backend and expands the impact of network interception, backend compromise, and accidental logging.

### Local Storage of Sensitive Wallet Material

The frontend stores DID and private-key-related material using React Native AsyncStorage.

This was identified through source-code review. Production iOS application storage was not extracted.

### OAuth / PKCE Randomness

Security-sensitive OAuth / PKCE values were generated using `Math.random()` in reviewed frontend code.

A cryptographically secure random source would provide stronger protection for these protocol values.

### Validation Hardening

Selected backend credential and presentation flows contained validation skip flags.

The code finding was confirmed, but the assessment did not prove that invalid credentials are accepted in production, so this was treated as a conditional hardening finding.

### Session Re-authentication

Face ID was required after a full application restart.

The wallet did not require fresh Face ID authentication after the tested backgrounding and phone lock / unlock sequence, which creates a local session-access risk.

### Backend Hardening

Controlled local tests identified several hardening concerns:

- Permissive CORS behavior
- Unauthenticated sensitive backend operations
- User-controlled outbound URL fetching
- No visible application-level throttling during local stress testing

These findings were treated as backend hardening observations because production infrastructure controls were outside the available assessment scope.

## Main Recommendations

The report recommends:

- Keep mnemonic handling, key derivation, and signing on the device where possible
- Protect private-key material using platform-secure storage such as iOS Keychain or Secure Enclave-backed keys
- Avoid transmitting private-key-related material to the backend
- Consider certificate pinning or equivalent mobile trust hardening
- Replace `Math.random()` with a cryptographically secure random source for OAuth / PKCE values
- Enable validation by default outside explicit test or conformance modes
- Restrict CORS and strengthen API abuse controls
- Restrict outbound URL fetching and block private or metadata-network targets
- Re-lock the wallet after backgrounding, device lock, or an inactivity timeout
- Improve wallet-specific privacy and destructive-action guidance

## Tools and Technologies

- Burp Suite Community
- iOS / iPhone testing
- React Native source-code review
- Node.js / Express backend analysis
- Python
- curl
- Qualys SSL Labs
- autocannon
- JWT / Verifiable Credential analysis
- OAuth / PKCE
- EBSI / SSI / Verifiable Credentials

## Skills Demonstrated

- Mobile application security testing
- Web and API security analysis
- HTTPS interception and traffic analysis
- Source-code security review
- Authentication testing
- Secure storage analysis
- OAuth / PKCE review
- CORS analysis
- SSRF-pattern identification
- Controlled availability testing
- Privacy review
- Threat modeling
- Security finding classification
- Remediation recommendations
- Ethical scoping and test-boundary definition

## Report

The full public portfolio report is included in this repository:

`portuguese-digital-wallet-security-analysis-public.pdf`

The public version redacts the live API hostname while preserving the assessment methodology, evidence, findings, limitations, and recommendations.

## Ethical Boundaries

All dynamic testing used my own test wallet data.

No other users' data was accessed. Production denial-of-service testing, brute force testing, broad fuzzing, and infrastructure attacks were not performed. Higher-risk experiments were reproduced only against a local backend instance.

## Disclaimer

This repository documents an academic security assessment. Findings are presented within the scope and limitations of the project and should not be interpreted as evidence of compromise of other users, production infrastructure, or systems outside the tested environment.
