# Extending Account Verification Handler
**Please note that this architecture is still a work in progress and might be subject to change. For further information, ask us on [gitter](https://gitter.im/gnosis/apollo).**

For legal compliancy, we have integrated a KYC provider who will verificate our users and give us confidence in their reputation. This integration was built with extendability for future verification handlers in mind.

Take a look at `src/verification/index` for a brief overview on how the interface handles verification integrations. You can look at the existing integration to derive a similar integration for your needs. A verification integration must fullfil the following criteria:
- Must have an entry point in `/src/verification/<name>/index.js` which returns a react component to give the user instructions.
- A verification needs to handle the verification status via the action-creator in `src/store/account`, namely `setVerificationStatus`, that should be used with the account and the name of your integration, like so: `setVerificationStatus(account, 'onfido')`.
- The verification will be embedded as a **Modal**, so be sure your verification can be viewed as such. If you require a more in-depth step-by-step integration, you need to create a `route` in `src/routes`, to which you can redirect the user when they start the verification.
- **This integration won't be enough to keep unauthorized users out of the system, it is your responsibility to integrate further safeguards in your backend or smart-contracts.**

