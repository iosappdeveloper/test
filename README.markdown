# Guidelines

1. Preferrably structure a test case function body in 3 sections -  given, when and then

Example:

```swift
func testSuccessUsingCompanyText() {
  // Given - valid company name 
  let companyText = "SAP"
  
  // When - get activation deeplink for given company
  SFActivationWebService.requestActivationDeeplink(forQuery: companyText) { response in
  
     // Then - response should be a success with proper activation deeplink url string
     if case let .success(activationUrl) = response {
        XCTAssertEqual(activationUrl, SFActivationWebServiceTest.successMap[companyText])
     } else {
        XCTFail("for company query text = \(companyText)")
     }
  }
}
```
