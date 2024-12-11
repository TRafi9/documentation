Specflow is used to conduct behavioural testing, giving non-technical users to write testing scenarios in the format:
Given situation X
When I do X
Then I expect X result

Each of these steps are then associated with functions the technical teams write to test these scenarios.

Feature files e.g. CreateSomething.Feature will include the simple wording written in gherkin syntax e.g.:
Given situation X
When I do X
Then I expect X result

The StepDefinitions files will then have each of those steps written out associated with functions e.g:

StepDefinitions.Given.cs file will have:
[Given(situation X)]
// some function for setting up the situation

StepDefinitions.When.cs file will have:
[When(I do X)]
// some function for executing the situation

3. Handle Common Setup via Background
   If the Given steps are the same or nearly identical across scenarios, you can use a Background section to avoid repetition:

gherkin
Copy code
Background:
Given the charity being uploaded contains the following fields:
| Charity Id | Parent Charity Id | Is National | Charity Name | Logo Uri | Charity Latitude | Charity Longitude | Postcode |
| 21ea65e5-e144-6b5f-cb1f-bfcf949faa1c | | false | Borer and Sons | | 37.051 | 156.4469 | PT8 6UC |
Then adjust scenarios as needed:

gherkin
Copy code
Scenario: Add charity with valid fields returns 201 Created
When I make a request to upload the new charity
Then I should get a 201 created response

Scenario: Add charity with invalid field types returns 400 Bad Request
Given the charity data has invalid field types
| Is National |
| yes |
When I make a request to upload the new charity
Then I should get a 400 bad request response
