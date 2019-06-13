## * This is a test
### This is a subtext
 - Point on why we should do things differently
 - Second point on why
+++
 
## Old way
+++
```
	@JL-31763 @v2.9+
	Scenario: Bulk purchase reject invalid file
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
+++
## New way
+++
```
@JL-31761 @v2.9+
    Scenario: Upload a valid CSV file with auth
        Given I have a "valid csv" file
        And I am an authenticated partner
        When I make the request
        Then I should see success messages
```
+++
