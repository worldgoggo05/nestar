# Understanding the `lookupAuthMemberLiked` Function

The `lookupAuthMemberLiked` function is a utility that creates a MongoDB aggregation pipeline stage for use in your NestAPI application. This function plays a crucial role in determining whether the currently authenticated user has "liked" a specific item (article, member, etc.) in your database.

## Function Overview

```typescript
export const lookupAuthMemberLiked = (memberId: T, targetRefId: string = '$_id') => {
    return {
        $lookup: {
            from: 'likes',
            let: {
                localLikeRefId: targetRefId,
                localMemberId: memberId,
                localMyFavorite: true
            },
            pipeline: [
                {
                    $match: {
                        $expr: {
                            $and: [
                                {$eq: ["$likeRefId", "$$localLikeRefId"]}, 
                                {$eq: ["$memberId", "$$localMemberId"]}
                            ],
                        },
                    },
                },
                {
                    $project: {
                        _id: 0,
                        memberId: 1,
                        likeRefId: 1,
                        myFavorite: "$$localMyFavorite",
                    },
                },
            ],
            as: "meLiked",
        },
    };
};
```
------------------------------------------------------------------------------------
## Purpose

This function creates a MongoDB `$lookup` aggregation stage that:

1. Joins (looks up) documents from the 'likes' collection
2. Checks if the current user (identified by `memberId`) has liked a specific item (identified by `targetRefId`)
3. Adds the matched like document(s) to the result as a field called `meLiked`

## How It Works

1. **Parameters**:
   - `memberId`: The ID of the currently authenticated user
   - `targetRefId`: The field containing the ID of the target item to check (defaults to '$_id')

2. **Pipeline Process**:
   - It creates variables (`let`) to use in the lookup pipeline
   - The `$match` stage finds documents where both the `likeRefId` matches the current item ID and the `memberId` matches the current user ID
   - The `$project` stage shapes the returned data, adding a `myFavorite: true` flag to indicate a match
   - Results are stored in the `meLiked` array field on each returned document

## Usage in Services

### 1. In `board-article.service.ts`

Used in the `getBoardArticles` method to determine if the current user has liked each article in the list. This enriches article data with personalized "liked" status before returning it to the client.



### 2. In `member.service.ts`

Used in the `getAgents` method to check if the current user has liked each agent member. This adds the "liked" status to agent data before returning it.


### 3. In `follow.service.ts`

Used in both `getMemberFollowings` and `getMemberFollowers` methods to check if the current user has liked each of the following/follower members. The function is used with a modified parameter to check against the appropriate ID field.


------------------------------------------------------------------------------------
## Benefits

1. **Reusability**: The function encapsulates a common MongoDB aggregation pattern, making it reusable across different services.

2. **Consistency**: Ensures that the "liked" status is determined in the same way throughout the application.

3. **Performance**: Uses MongoDB's aggregation pipeline for efficient joining of data rather than making separate queries.

4. **Personalization**: Enables you to show personalized UI elements (like filled/unfilled heart icons) based on whether the current user has liked an item.

## Summary

The `lookupAuthMemberLiked` function is a key utility in your application that allows you to efficiently determine and include the "liked" status of items for the current user, enhancing the user experience by providing personalized data in API responses.


           --------------------------------------------------------------------------------------------------
           ----------------------------------------------------Follow ---------------------------------------
# Understanding the `lookupAuthMemberFollowed` Function

The `lookupAuthMemberFollowed` function is another essential utility that creates a MongoDB aggregation pipeline stage for your NestAPI application. This function is crucial for determining whether the currently authenticated user is following a specific member in your database.

## Function Overview

```typescript
interface LookupAuthMemberFollowed {
    followerId: T;
    followingId: string;
};

export const lookupAuthMemberFollowed = (input: LookupAuthMemberFollowed) =>{
    const {followerId, followingId} = input;
    return {
        $lookup: {
            from: 'follows',
            let: {
                localFollowerId: followerId,
                localFollowingId: followingId,
                localMyFavorite: true
            },
            pipeline: [
                {
                    $match: {
                        $expr: {
                            $and: [{ $eq: ["$followerId", "$$localFollowerId"]}, {$eq: ["$followingId", "$$localFollowingId"] }],
                        },
                    },
                },
                {
                    $project: {
                        _id: 0,
                        followerId: 1,
                        followingId: 1,
                        myFollowing: "$$localMyFavorite",
                    },
                },
            ],
            as: "meFollowed",
        },
    };
};
```

## Purpose

This function creates a MongoDB `$lookup` aggregation stage that:

1. Joins (looks up) documents from the 'follows' collection
2. Checks if the current user (identified by `followerId`) is following a specific member (identified by `followingId`)
3. Adds the matched follow document(s) to the result as a field called `meFollowed`

## How It Works

1. **Parameters**:
   - Takes an `input` object containing:
     - `followerId`: The ID of the currently authenticated user
     - `followingId`: The field containing the ID of the target member to check

2. **Pipeline Process**:
   - It creates variables (`let`) to use in the lookup pipeline
   - The `$match` stage finds documents where both the `followerId` matches the current user ID and the `followingId` matches the target member ID
   - The `$project` stage shapes the returned data, adding a `myFollowing: true` flag to indicate a follow relationship exists
   - Results are stored in the `meFollowed` array field on each returned document

## Usage in Services

### In `follow.service.ts`

Used in both the `getMemberFollowings` and `getMemberFollowers` methods to determine if the current user is following each member in the results:

1. In `getMemberFollowings`: Checks if the authenticated user is following each of the members that a specific user is following
2. In `getMemberFollowers`: Checks if the authenticated user is following each of the members that follow a specific user

This enriches the member data with personalized "following" status before returning it to the client.

## Benefits

1. **Personalized UI**: Enables your application to show appropriate UI elements (like "Following" badges or follow/unfollow buttons) based on the current user's follow relationships.

2. **Efficient Queries**: Handles the follow status check within the database query rather than requiring additional queries or client-side processing.

3. **Consistent Pattern**: Follows the same pattern as the `lookupAuthMemberLiked` function, making the codebase more maintainable and easier to understand.

4. **Social Network Features**: Essential for implementing social network features where users can see and manage their follow relationships.

## Summary

The `lookupAuthMemberFollowed` function is a key utility in your application that allows you to efficiently determine and include the "following" status between the current user and other members. This enhances the user experience by providing personalized social relationship data in API responses.



------------------------------------------------------------------------------------------
## MongoDB $ Operatorlarini Tushunish

Yuqoridagi kodda `$` bilan boshlanadigan bir nechta MongoDB operatorlari ishlatilgan. Keling, ularning har birini ko'rib chiqamiz:

### 1. `$expr`
- Soʻrov ichida agregatsiya ifodalaridan foydalanishga imkon beradi
- Bir xujjatdagi maydonlarni taqqoslash imkonini beradi
- Bu yerda bir nechta maydon taqqoslashlarini birlashtirish uchun ishlatiladi

### 2. `$and`
- Bir nechta shartlarni birlashtiruvchi mantiqiy operator
- Xujjat mos kelishi uchun barcha shartlar "true" qiymatini qaytarishi kerak
- Bu holatda ikkita tenglik tekshiruvini birlashtiradi:
  - likeRefId mosligini
  - memberId mosligini

### 3. `$eq`
- Ikkita qiymatning tengligini tekshiruvchi taqqoslash operatori
- Massiv formatida ishlatiladi: `{$eq: ["$maydonNomi", "$$o'zgaruvchiNomi"]}`
- Bitta `$` prefiks joriy xujjatdagi maydonni bildiradi
- Ikkita `$$` prefiks pipelineda aniqlangan o'zgaruvchini bildiradi



