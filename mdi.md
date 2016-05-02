## Monitoring Driven Infrastructure (MDI)
As I've previously stated, theory is a type of tool. I would like to present a theory here, briefly, regarding how I intend to develop infrastructure going forward.

I like to call it Monitoring Driven Infrastructure, a phrase I've coined off of the back of [Test Driven Development](https://en.wikipedia.org/wiki/Test-driven_development) and one I like to use to describe how infrastructure should be developed. Simply put: I believe your monitoring solution should be developed and put in place first, with all checks and thresholds defined, followed by your actual infrastructure housing your application(s). Your application and all resources supporting it should be refined to turn your monitoring dashboard of choice from red to green.

This yields some some advantages:

* You can define what your infrastructure should look like based on business needs;
* Based on feedback from your monitoring, you know what needs to be done;
* When your infrastructure is done, it's being monitoring from the beginning;
* Your monitoring solution documents your infrastructure for you, especially if you use Terraform;

Perhaps it's a way of moving forward which will fall on its face, but I intend to try anyway throughout the guide.