## Behat tests
### Proposed changes
+++
## Current way
 -  @color[red](**-**)overly verbose
 -  @color[red](**-**)Steps can be redundant
 -  @color[red](**-**)Reads like a test
 -  @color[green](**+**)No custom context file creation
+++
```
@JL-31763 @v2.9+
Scenario: Bulk purchase accepts valid file
	Given I have a valid SKEY
		And I request "/v2/csv_upload" using HTTP GET
		And the response is success
		And the response code is 200			
	When the request body is:
			"""
			{
				"upload_csv": "invalid_file"
			}
			"""
```
@snap[east span-20]
## Part 1
@snapend
+++
```
	And I request "/v2/csv_upload" using HTTP POST
Then the response is success
	And the response code is 200
	And the response reason phrase is "OK"
	And the response body contains JSON:
		"""
		{
			"result": {
				"csv_validity": "@variableType(string)"
			}
		}
		"""
```
@snap[east span-20]
## Part 2
@snapend
+++
## Proposed way
- @color[red](**-**)More work
- @color[red](**-**)Steps can obfuscate the actions
- @color[green](**+**)Easier to read
- @color[green](**+**)Describes the functionality
- @color[green](**+**)Makes it clearer what is being tested
+++
```
@JL-31761 @v2.9+
Scenario: Upload a valid CSV file with auth
	Given I have a "valid csv" file
	And I am an authenticated partner
	When I send the "file" request
	Then I should see success messages
```
+++
## Custom Context
### Step definitions
- Methods from apiContext
- One custom method
+++
```
/**
 * @Given I am an authenticated partner
 */
public function iAmAnAuthenticatedPartner()
{
	$partnerArray = ["partner_id" => '1'];
	$partnerJSON = json_encode($partnerArray);
	$this->apiContext->setTestRequestFormParams($partnerJSON);
}
```
+++
```
/**
 * @Given I have a :file file
 */
public function iHaveAFile($file)
{
	switch ($file) {
		case "valid csv":
			$this->apiContext->addMultipartFileToRequestCustomName('/opt/behat/project/_mocks/test.csv', 'file', $this->iHaveSetupTheTestFiles());
			break;
		case "invalid doc":
			$this->apiContext->addMultipartFileToRequest('/opt/behat/project/_mocks/invalidDoc.doc', 'file');
			break;
		case "invalid missing headers csv":
			$this->apiContext->addMultipartFileToRequestCustomName('/opt/behat/project/_mocks/invalid_missing_headers.csv', 'file', 'missing_headers_' . microtime(true) . '.csv');
			break;
		case "valid csv duplicate":
			$this->apiContext->addMultipartFileToRequest('/opt/behat/project/_mocks/test.csv', 'file');
			break;
	}
}
```
+++
```
/**
 * @When I send/update the :path request
 */
public function iSendTheRequest($path)
{
	$this->apiContext->requestPath("/$path", 'POST');
}
```
+++
```
/**
 * @Then I should see success messages
 */
public function iShouldSeeSuccessMessages()
{
	$api_response = $this->apiContext->getResponseBody();
	$this->apiContext->assertResponseIs('success');
}
```
+++
## Context inheritance
+++?image=contextinheritance.png&size=auto 

