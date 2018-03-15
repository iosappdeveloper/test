# Downloaded online conent - Remove

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
