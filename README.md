## Curato Create and Update

### Update

1. Whenever media is added to S3, Lambda adds a message to SQS.
2. The handler method is called with the message object received from SQS. This handler method is in our media junction API (curator.ts).
3. The message object is validated, and some properties are extracted from it (workers/curator/index.ts).
4. The `mainHandler` method in `index.ts` is called.
5. Objects are uploaded to the S3 bucket (`tbn-dev-media-junction-api-assets`).
6. The `upsertCuratoMedia` method is called with some JSON received from SQS.
    1. It checks if the media is new or existing based on the update property (Rest/curator/index.ts).
    2. We receive `update = true` even if they are checking to hit the DB call again.
    3. If the record exists, the update functionality is executed.
        1. The `updateCuratorMedia` method is called, which calls another method:
            1. `getMediaByIdWithAdBreaks` returns the previous state.
                1. Retrieves `MediaItem` containing `VideoAsset`, `thumbnailAsset`, and `dynamicLink`.
                2. Retrieves `trailerQuery` containing `thumbnailAsset` and `mediaBasicInfo`.
                3. Retrieves `tenantSiteId` from the `TenantTable`.
                4. Retrieves `liveEventStatus` from the `liveEvents` table.
                5. Retrieves `mediaTransactionQuery` where `syncStatus` is started or pending.
                6. Retrieves `lastPublishedDetailsQuery` where `syncStatus` is completed.
                7. Retrieves `lastSyncTimeQuery` where `syncType` is `FullSync` or `PartSync`.
                8. Retrieves `lastAdBreakSyncTimeQuery` where `syncType` is `AdBreakSync`.
                9. Retrieves `transcodingStatus` based on `externalId` and `tenantId`.
                10. Prepares JSON with the above data and sends it back to the previous state.
            2. Merges old custom properties and new custom properties from the previous state.
            3. Updates media and `videoAsset` with the above JSON.
        2. Existing captions are deleted, and new captions are added.
7. The `finalizeLogDetails` method is called.
    1. Calls `finalizeImportedLog`.
    2. In the log, `publishMediaToJW` is executed.
    3. `publishMediaTransactionToSQS` is executed.




### Create

1. Whenever media is added to S3, Lambda adds a message to SQS.
2. The handler method is called with the message object received from SQS. This handler method is in our media junction API (curator.ts).
3. The message object is validated, and some properties are extracted from it (workers/curator/index.ts).
4. The `mainHandler` method in `index.ts` is called.
5. Objects are uploaded to the S3 bucket (`tbn-dev-media-junction-api-assets`).
6. The `upsertCuratoMedia` method is called with some JSON received from SQS.
    1. It checks if the media is new or existing based on the update property (Rest/curator/index.ts).
    2. We receive `update = true` even if they are checking to hit the DB call again.
    3. If the record does not exist, the create functionality is executed.
    4. The `createCuratorMedia` method is called.
        1. Extracts properties from the argument, which is a parameter of `createCuratorMedia`.
        2. Retrieves `networkLang` from the `networks` table where `tenantID`.
        3. Retrieves `mediaTypes` from the `mediaTypes` table where `tenantID`.
        4. Retrieves existing media IDs from the `media` table and gets all IDs with the object.
        5. Loops through all media and fetches existing codes from each object.
        6. Calls the `generateUniqueCode` method.
            1. Creates a new random ID and compares it with all existing codes.
            2. If no match is found, returns the new ID.
        7. Prepares some JSON.
        8. Calls the DB to create media with the prepared JSON.
        9. Retrieves `mediaToUpdate` from the created media.
        10. Calls the DB to create `mediaBasicInfo`.
        11. Calls the DB to create `thumbnailAsset`.
        12. Calls the DB to create `Caption`.
7. The `finalizeLogDetails` method is called.
    1. Calls `finalizeImportedLog`.
    2. In the log, `publishMediaToJW` is executed.
    3. `publishMediaTransactionToSQS` is executed.
