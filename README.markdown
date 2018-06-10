# Guidelines

1. Preferrably structure a test case function body in 3 sections -  given, when and then


```swift
func testSuccessUsingCompanyText() {
  // Given - valid company name 
  let companyText = "SAP"
  ...
  // When - get activation deeplink for given company
  SFActivationWebService.requestActivationDeeplink(forQuery: companyText) { response in
     // Then - response should be a success with proper activation deeplink url string
     if case let .success(activationUrl) = response {
        XCTAssertEqual(activationUrl, SFActivationWebServiceTest.successMap[companyText])
     } else {
        XCTFail("for company query text = \(companyText)")
     }
     ...
  }
  ...
}
```

2. Test Asynchronous operations using XCTestExpectation


```swift
func testSuccessUsingCompanyText() {
  // Given - valid company name 
  let companyText = "SAP"
  
  let promise = expectation(description: "Call to web service")
  // When - get activation deeplink for given company
  SFActivationWebService.requestActivationDeeplink(forQuery: companyText) { response in
     // Then - response should be a success with proper activation deeplink url string
     if case let .success(activationUrl) = response {
        XCTAssertEqual(activationUrl, SFActivationWebServiceTest.successMap[companyText])
     } else {
        XCTFail("for company query text = \(companyText)")
     }
     promise.fulfill()
  }
  wait(for: [promise], timeout: 3)
}
```

3. Fake network response using MockNetworkManager (Dependency injection)

```swift
    func testAICCWrapperOnlineSettings() {
        // Initialize online service with a mock network manager
        let aiccWrapperService = AICCWrapperOnlineService(networkManager: MockNetworkManager())
        aiccWrapperService.performFetch()
        let expection = expectation(description: "Online fetch update handler")
        aiccWrapperService.fetchUpdateHandler = {
            XCTAssertNotNil(aiccWrapperService.fetchResult)
            XCTAssert(aiccWrapperService.agreeText?.isEmpty == false)
            XCTAssert(aiccWrapperService.disagreeText?.isEmpty == false)
            XCTAssert(aiccWrapperService.instructionText?.isEmpty == false)
            expection.fulfill()
        }
        wait(for: [expection], timeout: 3)
    }

fileprivate extension AICCWrapperTest {
    class MockNetworkManager: SFNetworkManager {
        override func execute(_ request: SFNetworkRequest!, success: SuccessFactorsCommon.SFNetworkSuccessBlock!, failure: SuccessFactorsCommon.SFNetworkFailureBlock!) {
            DispatchQueue.main.async {
                let url = Bundle(for: type(of: self)).url(forResource: "aiccWrapperSettings", withExtension: "json")!
                guard let data = try? Data(contentsOf: url) else {
                    success(nil, nil)
                    return
                }
                success(nil, data)
            }
        }
    }
}
```
