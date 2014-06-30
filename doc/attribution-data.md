## In-app attribution data access

Our SDKs offer a facility to allow you to pull attribution and source data from our servers into your app, so that you can customize your app and leverage the data to provide source-specific features or onboarding flows.

If you use this facility, please note that in-app attribution data access is designed to forward the data to you, in a similar manner as server-to-server callbacks. Data which is transmitted to you should be handled with care as it may be confidential under agreements that you may have with e.g. traffic partners, such as data covered by the Facebook Advanced Mobile App Tracking Terms and Conditions. adjust can not take responsibility for your usage of data which has been transmitted to you via these interfaces. 

Specifically, if you are working with Facebook data and are looking to forward attribution data in the SDK to other partners such as Mixpanel, you should filter the detailed Facebook data (with ad group and campaign IDs) to a single "Facebook" label, which does not include ad group IDs. Strictly no third parties should have access to Facebook targeting data in association with device IDs.

Feel free to reach out to support@adjust.com if you have any questions.
