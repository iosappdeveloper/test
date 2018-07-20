# Guidelines

1. Write XCTestCase for one object and test one behavior at a time
   * One XCTestCase file for every object/system under test (SUT)
   * One behavior of one component at a time
   * Prefer to structure unit test function body in 3 sections -

      * Given - initial conditions

      * When - perform actions

      * Then - expect output/states

      Note, many a times `setup()` function would have some of the intial conditions or even all

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
   * Usually an object (system under test i.e. SUT) depends on other objects to perform a behavior. Some of these dependencies are external or flaky in nature e.g network operations, UserDefaults, persistent store etc.
   * Dependency injection in a SUT object can be done through -
      + constructor
      + property
      + function

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

4. Strive to write one `XCTestCase` class(file) to test only one component (SUT), not multiple.
    * name of test class/file should include the test component's name. For example -
      * actual component's name: `LMSOfflineContentBrowserViewModel`
      * test component's name: `LMSOfflineContentBrowserViewModelTest`
    * if you need to create another component to test the selected component then its perfectly fine (integrated testing).
    
5. Prefer to write one test function to test only a single behavior of test component

6. Prefer to use swift language error handling mechanism to catch programmer/compiler's error of test code. This helps in writing non-distracting and clean code in test.
    * avoid test framework statements like `XCTFail`/`XCTAssert` or even `fatalError()` to cover errors that are bound to be programmer or compile time (language) errors in actual test.
    ```swift
    func testRemoveOfflineContent() throws {
        // given
        try prepareContentBrowserViewModel(jsonFileName: "offlineStudentContents")
        let launchFilePath = "\(learningItemCpntKey)/test.jabber"
        let absoluteFilePath = try createSampleLaunchFile(atContentLocation: launchFilePath)
        XCTAssertTrue(FileManager.default.fileExists(atPath: absoluteFilePath.path))
        // when
        contentBrowserViewModel?.removeContent()
        // then
        XCTAssertFalse(FileManager.default.fileExists(atPath: absoluteFilePath.path))
    }

    ```
    
    * test function itself can `throws` error outside to XCTest framework. This allows propogation of such errors.
    * generally statements in `given` section including test's helper functions are fine to `throw` or `throws` errors
    * avoid over use of `throw` especially in `when` and `then` section as those are actually meant to assert the outcome
  
