# Downloaded online content - Remove

Show confirmation UI and proceed with remove handling only if user provides consent. 

### Check sync status of this item's progress
### If progress is already synced 
1. Remove downloaded content from file system
2. Remove progress and other offline entities
3. Take user out of content structure screen

### If progress is NOT synced
1. Remove downloaded content file system and persistent store data (offline content and url mapping) **except** progress data stored in `StudentComponent` + `StudentComponentMod`
2. Queue content sync on background thread (async)
3. Take user out of content structure screen

Note - Learning plan API is a required follow up API call of content sync API in sync service framework


# Offling Content Settings - Pass/Fail

For a course, if learning admin user has enabled (set) offline content settings i.e. pass or fail checked along with optional failure action/retry, then once learning user passes or fails a module (content object) then do not allow user to launch any content object in that course

1. Read pass and fail boolean values of content settings from `triggerStudentComponentPass` and `triggerStudentComponentFail` keys respectively in the response of existing current-user/downloadonlinecomponent API
2. For each module (content object) store both these values in `StudentComponentMod` entity

```swift
@interface StudentComponentMod : NSManagedObject
...
...
@property (nonatomic, retain) NSNumber * triggerStudentComponentPass;
@property (nonatomic, retain) NSNumber * triggerStudentComponentFail;
@end
```

3. Every time we reload content structure screen (push or pop) then loop through all modules and disable launch module action for all modules in that course if one of these 2 conditions is true for any _one_ module -
   1. `triggerStudentComponentFail` is enabled and user has `finished` but not `complete`d a module
   2. `triggerStudentComponentPass` is enabled and user has `complete`d a module

```swift
            // Check if status has been recorded for ANY content object, passed or failed, since user is not allowed to access other content objects once an event is recoreded for one
            for studentComponentModule in (studentComponent.studComponentMod as! Set<StudentComponentMod>) {
                if studentComponentModule.triggerStudentComponentFail.boolValue, studentComponentModule.finished.boolValue, !studentComponentModule.complete.boolValue {
                    recordedStatus = .fail
                    break
                } else if studentComponentModule.triggerStudentComponentPass.boolValue, studentComponentModule.complete.boolValue {
                    recordedStatus = .pass
                    break
                }
            }
```

Note - it is possible that multiple modules in a course has pass/fail settings but once the user pass or fail one such module then it is considered client code's responsiblity to block the course from being taken by the user. Hence, it can be assumed that at a time only one such module in a course will pass or fail.

Further, learning admin can configure a course to be taken multiple times (>=1 retries) with 2 failure actions after all attempts are taken by user - lock item or remove item from user's learning plan

In both cases, the content structure or rendering screens does not do anything special. However, once user comes out of content screen and the progress is synced with server then -
1. In response of currentUserLearningPlan API call, we check if studentComponentID doesn't match with the one we have stored the previous progress then simply deleted the offline content.
   1. Note we need to store `componentKey` in `StudentComponent` entity and `studentComponentID` in `LearningItemData` entity to identify this mismatch.
   
   ```swift
           if (learningItemData.studentComponentID.integerValue != studentComponent.studentComponentID.integerValue) {
              [learningItemData removeOfflineContent];
           }
   ```
   
2. In case failure action is lock item then we read an additional key `onlineStatus` inside `statusVOX` of each userTodo in `currentUserLearningPlan`. If this value is -2 then we can show **Locked out** status.
