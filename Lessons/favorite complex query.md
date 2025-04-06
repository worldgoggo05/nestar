# Understanding the `lookupFavorite` and `lookupVisited` Functions

These utility functions create MongoDB aggregation pipeline stages used in the Nestar API application to enhance property data with member information when retrieving favorite or visited properties.

## Function Overview

```typescript
export const lookupFavorite = {
    $lookup: {
        from: 'members',
        localField: 'favoriteProperty.memberId',
        foreignField: '_id',
        as: 'favoriteProperty.memberData',
    },
};

export const lookupVisited = {
    $lookup: {
        from: 'members',
        localField: 'visitedProperty.memberId',
        foreignField: '_id',
        as: 'visitedProperty.memberData',
    },
};
```

## Purpose
------------------------------------------------------------------------------------
These functions create MongoDB `$lookup` aggregation stages that:

1. Join (look up) documents from the 'members' collection
2. Connect property data with the details of the member who created the property
3. Enrich the property data with member information for display in the UI

## How They Work

### `lookupFavorite`
- **From**: Looks up documents in the 'members' collection
- **LocalField**: Uses 'favoriteProperty.memberId' as the field from the input documents to match against
- **ForeignField**: Matches against the '_id' field in the 'members' collection
- **As**: Stores the matched member document in a nested field 'favoriteProperty.memberData'

### `lookupVisited`
- **From**: Looks up documents in the 'members' collection
- **LocalField**: Uses 'visitedProperty.memberId' as the field from the input documents to match against
- **ForeignField**: Matches against the '_id' field in the 'members' collection
- **As**: Stores the matched member document in a nested field 'visitedProperty.memberData'

## Usage in Services
------------------------------------------------------------------------------------
### In `like.service.ts`

Used in the `getFavoriteProperties` method to enrich favorite property data with information about the property creator:

```typescript
public async getFavoriteProperties(memberId: ObjectId, input: OrdinaryInquiry): Promise<Properties> {
    const {page, limit} = input;
    const match: T = {likeGroup: LikeGroup.PROPERTY, memberId: memberId};

    const data: T = await this.likeModel.aggregate([
        {$match: match},
        {$sort: {updatedAt: -1}},
        {
            $lookup: {
                from: 'properties',
                localField: 'likeRefId',
                foreignField: '_id',
                as: 'favoriteProperty',
            },
        },
        {$unwind: '$favoriteProperty'},
        {
            $facet: {
                list: [
                    {$skip: (page - 1) * limit},
                    {$limit: limit},
                    lookupFavorite,
                    {$unwind: '$favoriteProperty.memberData'},
                ],
                metaCounter: [{$count: 'total'}],
            }
        }
    ])
    .exec();
    const result: Properties = {list: [], metaCounter: data[0].metaCounter};
    result.list = data[0].list.map((ele) => ele.favoriteProperty);
    return result;
}
```
------------------------------------------------------------------------------------

### In `property.service.ts`

The `getFavorites` method delegates to the LikeService's `getFavoriteProperties` method:

```typescript
public async getFavorites(memberId: ObjectId, input: OrdinaryInquiry): Promise<Properties> {
    return await this.likeService.getFavoriteProperties(memberId, input);
}
```

### In `property.resolver.ts`

The resolver exposes the functionality to GraphQL, making it available to the frontend:

```typescript
@UseGuards(AuthGuard)
@Query((returns) => Properties)
public async getFavorites(
    @Args('input') input: OrdinaryInquiry, 
    @AuthMember('_id') memberId: ObjectId
): Promise<Properties> {
    console.log('Query: getFavorites');
    return await this.propertyService.getFavorites(memberId, input);
}
```
------------------------------------------------------------------------------------
## The `OrdinaryInquiry` Type

This input type provides pagination parameters for the queries:

```typescript
@InputType()
export class OrdinaryInquiry {
    @IsNotEmpty()
    @Min(1)
    @Field(() => Int)
    page: number;

    @IsNotEmpty()
    @Min(1)
    @Field(() => Int)
    limit: number;
}
```

## Benefits

1. **Efficient Data Retrieval**: Combines favorite/visited properties with their creator information in a single database query
2. **Optimized Pagination**: Works with the pagination parameters to efficiently fetch just the required properties
3. **Complete Property Information**: Ensures property data includes details about the member who created it
4. **Consistent Pattern**: Uses the same aggregation pattern across the application
5. **Performance**: Avoids multiple database queries by using MongoDB's aggregation pipeline

## Summary

The `lookupFavorite` and `lookupVisited` functions are important utilities that enhance the property data retrieved from the database with creator information. This creates a more complete data structure for the frontend, allowing it to display not just property details but also information about the property creators without requiring additional API calls.


------------------------------------------------------------------------------------
## The `@Min(1)` Decorator

The `@Min(1)` decorator is a validation decorator from the `class-validator` library that ensures a numeric value is not less than the specified minimum (1 in this case). 

### Purpose

1. **Input Validation**: Prevents negative or zero values from being passed as pagination parameters
2. **Data Integrity**: Ensures that page and limit values make logical sense (you can't have page 0 or fetch 0 items)
3. **Error Prevention**: Catches invalid input early before it reaches the database query

### How It Works

When a request comes in with an `OrdinaryInquiry` object:

1. The validation pipe checks all decorated properties
2. If `page` or `limit` is less than 1, validation fails
3. A validation error is thrown before the request reaches the resolver
4. The client receives a clear error message about the invalid input

### Example Error

If a client tries to request page 0:
