---
title: API Reference

language_tabs: # must be one of https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers
  - graphql
  - javascript
  - jsx
  - json

toc_footers:
  - <a href='mailto:kay@heartfulsprout.com'>Contact Kay for any questions!</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Kittn API
---

# Introduction

Welcome to the Heartful Sprout API! You can use our API to access our API endpoints.

This documentation is primarily built for graphQL APIs served through Hasura. You can view the example queries in the dark area to the right.

If there are other languages in the workflow, you can switch the programming language of the examples with the tabs in the top right.


# Authentication in REST API

> To authorize as admin, use this code (javascript):

```javascript
export async function callHasuraQuery(
   gql: string,
   operationName: string,
   variables: any
) {
   const options = {
       "next": { "revalidate": 3600 },
       "method": "POST",
       "headers": {
           "content-type": "application/json",
           "X-hasura-admin-secret": process.env.NEXT_PUBLIC_HASURA_ADMIN_SECRET!
       },
       "body": JSON.stringify({
           "operationName": operationName,
           "query": gql,
           "variables": variables
       })
   }
   const res = await fetch(process.env.NEXT_PUBLIC_HASURA_PROJECT_ENDPOINT!, options)
   const data = await res.json();
   return data;
}

```

> To authorize as user, use this code (Javascript):

```javascript
export async function callHasuraQuery(
   gql: string,
   operationName: string,
   variables: any
) {
   const options = {
       "next": { "revalidate": 3600 },
       "method": "POST",
       "headers": {
           "content-type": "application/json",
           "Authorization": `Bearer ${userIdToken}`
       },
       "body": JSON.stringify({
           "operationName": operationName,
           "query": gql,
           "variables": variables
       })
   }
   const res = await fetch(process.env.NEXT_PUBLIC_HASURA_PROJECT_ENDPOINT!, options)
   const data = await res.json();
   return data;
}

```


Hasura is used for a simple data fetch and data update. It has only one endpoint with different graphQL commands. How to call the endpoint is written on the right, and graphQL commands can be found in this document.

Note that environment variables are not explicitly written in this documentation. 

<aside class="notice">
API calls require either admin secret or user token uses API keys to allow access to the API. Please let Kay know if you need admin secret.
</aside>


The userIDToken is automatically generated in jwt format when authenticated through AWS Cognito. The AWS Cognito returns accessToken and idToken, and the idToken is the token to be used for Hasura. It should contain:

* role: user
* X-Hasura-User-ID: <code>user ID from Cognito</code>


# Apollo Provider (React Native)

## Create an Apollo Client

The example code here is in jsx format. Please click jsx tab on the top right tab menu.
> Create apollo.js file:

``` jsx
import { ApolloClient, InMemoryCache, HttpLink } from "@apollo/client";
import {HASURA_ENDPOINT} from '@env';

export const createApolloClient = (authToken) => {
 return new ApolloClient({
   link: new HttpLink({
     uri: HASURA_ENDPOINT,
     headers: {
       'Authorization': `Bearer ${authToken}`
     },
   }),
   cache: new InMemoryCache(),
 });
};

```


## Use Apollo Provider

> Wrap around the MAin Navigator with the ApolloProvider

```jsx
  <NavigationContainer linking={linkingConfig}>
     {token === null ? <AuthNavigator /> : (
       <ApolloProvider client={createApolloClient(token?.getJwtToken())}>
         <MainNavigator token={token} />
       </ApolloProvider>
     )}
   </NavigationContainer>
```


## useQuery
useQuery is a hook. Please select jsx to see the example codes on the right.

```jsx
import {GQL_COMMENT} from '@env';

const {data, error} = useQuery(GQL_COMMAND);

if (data){
  processDataHere();
}
```


## useMutation
useMutation is a hook. Please select jsx to see the example codes on the right.

```javascript
import {GQL_COMMENT} from '@env';

const [gqlFunction, {data, error}] = useMutation(GQL_COMMAND);

gqlFunction({
  variables: inputParams
})

if (data){
  processDataHereIfNeeded();
}
```



# ----- B2C

# App Loading

We load as much as user-specific information up-front when app is being loaded once authenticated. The data fetch is completely based on the user email address that passed the authentication.


## B2C full data fetch (mobile app)

```graphql
query fetchSetupData($email: String!, $start_at: timestamp!) {
  user_account(where: {email: {_eq: $email}}) {
    id
    uid
    email
    has_profile
    nickname
    goal
    bio
    link
    user_type
    thumbnail_link
    reminder_freq_id
    cooking_freq_id
    for_who
    adventure
    repeatMenu
    baby_profiles {
      id
      name
      birthday
      baby_growth_hists(limit: 6, order_by: {days: desc}) {
        growth_type_id
        percentile
        value
        days
      }
      sex_type_id
      baby_ing_intros {
        ing_intro {
          name
          ingredients {
            food_group_id
          }
        }
        attempted_at
        mastered_at
      }
    }
    user_tags(where: {removed_at: {_is_null: true}}) {
      tag_id
    }
    user_diet_restrictions{
      diet_type
    }
    user_food_allergies{
      food_allergy_type
    }
    user_follows(where: {removed_at: {_is_null: true}}, distinct_on: follower_id) {
      follower_id
    }
    userFollowsByFollowerId(where: {removed_at: {_is_null: true}}, distinct_on: followee_id) {
      followee_id
    }
    recipe_hists(where: {removed_at: {_is_null: true}, planned_at: {_gte: $start_at}}) {
      id
      planned_at
      served_at
      served_amount
      reaction_id
      leftover
      note
      recipe {
        id
        title
        link_pic
        base_serving_num
        user_account {
          nickname
        }
        recipe_ings {
          ingredient {
            food_group_id
          }
        }
        recipe_nutritions {
          added_sugar_G
          calcium_MG
          calories_KCAL
          carbohydrate_G
          cholesterol_MG
          choline_MG
          folatedfe_UG
          iron_MG
          magnesium_MG
          niacin_MG
          omega3_G
          omega6_G
          phosphorus_MG
          potassium_MG
          protein_G
          riboflavin_MG
          saturated_fatty_acids_G
          sodium_MG
          thiamin_MG
          total_dietary_fiber_G
          total_fatty_acids_G
          total_lipid_G
          total_sugar_G
          trans_fatty_acids_G
          vitamina_UG
          vitaminb12_UG
          vitaminb6_MG
          vitaminc_MG
          vitamine_MG
          vitamind_UG
          vitamink_UG
          zinc_MG
        }
      }
    }
    recipes_aggregate(where: {recipe_status_id: {_eq: COMPLETE}}) {
      aggregate {
        count
      }
    }
    user_milestones {
      milestone_id
    }
    baby_reaction_hists(
      order_by: {updated_at: asc}, 
      where: {removed_at: {_is_null: true}}
    ) {
      id
      entered_at
      reaction
      note
    }
  }
  recipe(order_by: {updated_at: desc}, where: {_and: [
    {removed_at: {_is_null: true}}, 
    {recipe_status_id: {_eq: COMPLETE, _neq: ARCHIVE}},
    {_or: [{user_saved_recipes: {user_account: {email: {_eq: $email}}}}, {user_account: {email: {_eq: $email}}}]}]}) {
    id
    user_account {
      uid
      nickname
      thumbnail_link
    }
    link_pic
    title
    description
    base_serving_num
    recipe_tags {
      tag_id
    }
    created_at
    updated_at
    comments(where: {removed_at: {_is_null: true}}, distinct_on: commenter_id) {
      user_account {
        uid
        nickname
      }
      comment
    }
    likes_aggregate {
      aggregate {
        count(distinct: true, columns: liker_id)
      }
    }
    likes(where: {removed_at: {_is_null: true}, user_account: {email: {_eq: $email}}}) {
      user_account {
        uid
      }
    }
  }
  ing_intro(where: {ingredients: {recipe_ings: {recipe: {recipe_hists: {user_account: {email: {_eq: $email}}}}}}}) {
    name
    allergen_type
  }
}


```

> Example input parameter:

```graphql
{
  email: "email@email.com",
  start_at: "2024-11-01"
}
```


Mobile app is fetching all the info that is relevant. See web version below for web MVP.

This query is limited to the specific user by automatically matching <code>user_account.uid</code> to idToken in authenticated jwt token.

### Parameters:

Parameter | Type | Description
--------- | ----------- | -----------
email     | text | ID from baby_profile table
start_at  | timestamp | data load look back date. E.g. if entered '2024-11-01', it fetches meal plan from '2024-11-01'.



### Example input parameters:

<code>
{
  email: "email@email.com",
  start_at: "2024-11-01"
}
</code>


## B2C mvp data fetch (web)

```graphql
query fetchSetupData($email: String!, $start_at: timestamp!) {
  user_account(where: {email: {_eq: $email}}) {
    id
    uid
    email
    has_profile
    nickname
    goal
    bio
    link
    user_type
    thumbnail_link
  }
  user_tags(where: {removed_at: {_is_null: true}}) {
    tag_id
  }
  user_diet_restrictions{
    diet_type
  }
  user_food_allergies{
    food_allergy_type
  }
  user_follows(where: {removed_at: {_is_null: true}}, distinct_on: follower_id) {
    follower_id
  }
  userFollowsByFollowerId(where: {removed_at: {_is_null: true}}, distinct_on: followee_id) {
    followee_id
  }
  recipe_hists(where: {removed_at: {_is_null: true}, planned_at: {_gte: $start_at}}) {
    id
    planned_at
    served_at
    recipe {
      id
      title
      link_pic
      base_serving_num
      user_account {
        nickname
      }
      recipe_ings {
        ingredient {
          food_group_id
        }
      }
    }
  }
  recipe(order_by: {updated_at: desc}, where: {_and: [
    {removed_at: {_is_null: true}}, 
    {recipe_status_id: {_eq: COMPLETE, _neq: ARCHIVE}},
    {_or: [{user_saved_recipes: {user_account: {email: {_eq: $email}}}}, {user_account: {email: {_eq: $email}}}]}]}) {
    id
    user_account {
      uid
      nickname
      thumbnail_link
    }
    link_pic
    title
    description
    base_serving_num
    recipe_tags {
      tag_id
    }
    created_at
    updated_at
    comments(where: {removed_at: {_is_null: true}}, distinct_on: commenter_id) {
      user_account {
        uid
        nickname
      }
      comment
    }
    likes_aggregate {
      aggregate {
        count(distinct: true, columns: liker_id)
      }
    }
    likes(where: {removed_at: {_is_null: true}, user_account: {email: {_eq: $email}}}) {
      user_account {
        uid
      }
    }
  }
  ing_intro(where: {ingredients: {recipe_ings: {recipe: {recipe_hists: {user_account: {email: {_eq: $email}}}}}}}) {
    name
    allergen_type
  }
}
```

This query is limited to the specific user by automatically matching <code>user_account.uid</code> to idToken in authenticated jwt token.

### Parameters

Parameter | Type | Description
--------- | ----------- | -----------
email     | text | user's email
start_at  | timestamp | data load look back date. E.g. if entered '2024-11-01', it fetches meal plan from '2024-11-01'.


### Example input parameters:

<code>
{
  email: "email@email.com",
  start_at: "2024-11-01"
}
</code>


<aside class="notice">
Note that it is pulling recipes that includes this user's recipes and saved recipes. And you need to parse whether the recipe is this user's own or someone elses (e.g. saved) manually based on the author's ID.
</aside>


> Example input parameter:

```graphql
{
  email: "email@email.com",
  start_at: "2024-11-01"
}
```


# Account


## Insert Account

```graphql
mutation InsertUserAccount($email: String!, $uid: String!, $nickname: String!, $thumbnailLink: String!) {
  insert_user_account_one(
    object: {
      email: $email, nickname: $nickname, uid: $uid, user_status: ACTIVE_FREE, thumbnail_link: $thumbnailLink
    }, 
    on_conflict: {
      constraint: user_account_uid_key, update_columns: [thumbnail_link, uid]
    }
  ) {
    id
    nickname
  }
}
```

> Example input parameters:

```graphql
{
  email: "email@email.com",
  uid: "a1c5a247-b70d-4c0c-9f84-9f6bd5243391"
}
```

AWS Cognito authenticates users, but we keep track of their data separately. Thus, when user creates an account through Cognito, we need to make sure that it gets inserted into our <code>user_account</code> data table. And because the user doesn't have a record in our database yet, we allow the row to be inserted without restriction.

### Parameters

Parameter | Type        | Description
--------- | ----------- | -----------
email     | text        | the user's email
uid       | uuid        | uid provided by Cognito
nickname  | text        | the user's nickname. It's the first part from email address by default in mobile app logic
thumbnailLink | text    | S3 key pointing to this user's thumbnail.

### Example input parameters
<code>
{
  email: "email@email.com",
  uid: "a1c5a247-b70d-4c0c-9f84-9f6bd5243391"
}
</code>

<aside class="notice">
From the front end, nickname is automatically extracted from the user's email address. And if there is a duplicate nickname, we automatically append 4-digit uuid. For example, the front end tries to save "email" as nickname from "email@email.com", and if that fails because there is a duplicate, it tries again with nickname "email-a12b".
</aside>

## Reactivate Account

```graphql
mutation ReactivateAccount($email: String!, $uid: String!) {
  update_user_account(where: {email: {_eq: $email}}, 
    _set: {user_status: ACTIVE_FREE, uid: $uid}){
    returning {
      id
      uid
      nickname
    }
  }
}

```

This is called when the uid from Cognito doesn't match with the uid from our database. This is necessary because otherwise, user token is not authorized.



## Deactivate Account
```graphql
mutation DeactivateAccount($user_account_id: uuid!) {
  update_user_account(where: {id: {_eq: $user_account_id}}, 
    _set: {user_status: DEACTIVATED}){
    returning {
      id
    }
  }
}
```


## Permanently Delete
```graphql
mutation DeleteAccount($user_account_id: uuid!) {
  delete_baby_growth_hist(where: {baby_profile: {user_account_id: {_eq: $user_account_id}}}) {
    affected_rows
  }
  delete_baby_ing_intro(where: {baby_profile: {user_account_id: {_eq: $user_account_id}}}) {
    affected_rows
  }
  delete_baby_reaction_hist(where: {user_account_id: {_eq: $user_account_id}}) {
    affected_rows
  }
  delete_baby_profile(where: {user_account_id: {_eq: $user_account_id}}) {
    affected_rows
  }
  delete_comments(where: {_or: [
    {commenter_id: {_eq: $user_account_id}},
    {recipe: {author_id: {_eq: $user_account_id}}}
  ]}) {
    affected_rows
  }
  delete_likes(where: {_or: [
    {liker_id: {_eq: $user_account_id}},
    {recipe: {author_id: {_eq: $user_account_id}}}
  ]}) {
    affected_rows
  }
  delete_user_follow(where: {_or: [
    {followee_id: {_eq: $user_account_id}}, 
    {follower_id: {_eq: $user_account_id}}]}) {    
    affected_rows
  }
  delete_search_record(where: {user_account_id: {_eq: $user_account_id}}) {
    affected_rows
  }
  delete_flag(where: {reporter_id: {_eq: $user_account_id}}) {
    affected_rows
  }
  delete_user_milestone(where: {user_account_id: {_eq: $user_account_id}}) {
    affected_rows
  }
  delete_user_saved_recipe(where: {_or:[
    {user_account_id: {_eq: $user_account_id}}, 
    {recipe: {author_id: {_eq:$user_account_id}}}
  ]}) {
    affected_rows
  }
  delete_user_tag(where: {user_account_id: {_eq: $user_account_id}}) {
    affected_rows
  }
  delete_user_diet_restriction(where: {user_account_id: {_eq: $user_account_id}}) {
    affected_rows
  }
  delete_user_food_allergy(where: {user_account_id: {_eq: $user_account_id}}) {
    affected_rows
  }
  delete_recipe_hist(where: {_or: [{user_account_id: {_eq: $user_account_id}}, {recipe: {author_id: {_eq: $user_account_id}}}]}) {
    affected_rows
  }
  delete_recipe_ing(where: {recipe: {author_id: {_eq: $user_account_id}}}) {
    affected_rows
  }
  delete_recipe_nutrition(where: {recipe: {author_id: {_eq: $user_account_id}}}) {
    affected_rows
  }
  delete_recipe_step(where: {recipe: {author_id: {_eq: $user_account_id}}}) {
    affected_rows
  }
  delete_recipe_tag(where: {recipe: {author_id: {_eq: $user_account_id}}}) {
    affected_rows
  }
  delete_recipe(where: {author_id: {_eq: $user_account_id}}) {
    affected_rows
  }
  delete_user_account(where: {id: {_eq: $user_account_id}}) {
    returning {
      id
    }
  }
}
```



# User Profile

## Create User Profile

```graphql
mutation CreateProfile(
  $baby_name: String
  $bday: String
  $sex: sex_type_enum
  $weight_value: Float
  $weight_perc: Float
  $height_value: Float
  $height_perc: Float
  $bmi_value: Float
  $bmi_perc: Float
  $days: Int
  $cooking_level: complexity_type_enum
  $goal: String
  $for_who: String
  $adventure: String
  $repeatMenu: String
  $source: String
  $user_account_id: uuid
  $tags: [user_tag_insert_input!]!
  $diet: [user_diet_restriction_insert_input!]!
  $allergy: [user_food_allergy_insert_input!]!
) {
  update_user_account(
    _set: {
      cooking_level: $cooking_level, has_profile: true, 
      goal: $goal, for_who: $for_who, adventure: $adventure, 
      repeatMenu: $repeatMenu, source: $source},
    where: {id: {_eq: $user_account_id}}
  ) {
    returning {
      id
    }
  }
  insert_baby_profile(
    objects: {
      name: $baby_name
      sex_type_id: $sex
      birthday: $bday
      user_account_id: $user_account_id
      baby_growth_hists: {
        data: [
          {
            growth_type_id: WEIGHT
            value: $weight_value
            percentile: $weight_perc
            days: $days
          }
          {
            growth_type_id: HEIGHT
            value: $height_value
            percentile: $height_perc
            days: $days
          }
          {
            growth_type_id: BMI
            value: $bmi_value
            percentile: $bmi_perc
            days: $days
          }
        ]
      }
    }
  ) {
    returning {
      id
    }
  }
  insert_user_tag(objects: $tags) {
    returning {
      id
    }
  }
  insert_user_diet_restriction(objects: $diet){
    returning {
      id
    }
  }
  insert_user_food_allergy(objects: $allergy){
    returning {
      id
    }
  }
}
```

This part is usually created during onboarding process.

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
user_account_id | uuid      | the user's database-generated ID
baby_name     | text        | baby's name
bday          | text        | baby's birthday
days          | integer     | age in days. This is calculated from the app
sex           | enum        | either 'MALE' or 'FEMALE'
weight_value  | float       | weight measure in kg
weight_perc   | integer     | weight percentile for the baby's age. If 50%, data format is 50.
height_value  | float       | height measure in cm
height_perc   | integer     | height percentile for the baby's age. If 50%, data format is 50.
bmi_value     | float       | bmi measure
bmi_perc      | integer     | bmi percentile for the baby's age. If 50%, data format is 50.
cooking_level | enum        | EASY, EASY_MEDIUM, MEDIUM, MEDIUM_COMPLEX
goal          | text        | goal that the user chooses
for_who       | enum        | CHILD or FAMILY
adventure     | enum        | Y, M, or N
repeatMenu    | enum        | Y or N
source        | enum        | PRACTITIONERS, FRIENDS_FAMILIES, SOCIAL_MEDIA, or OTHERS
tags          | list        | list of {user_account_id: user_account_id, tag: TAG_CODE}. See Tag list below.
diet          | list        | list of {user_account_id: user_account_id, diet_type: DIET_TYPE_CODE}. See Diet Type list below.
allergy       | list        | list of {user_account_id: user_account_id, food_allergy_type: ALLERGY_TYPE_CODE}. See food allergy code list below.

### Example input parameters

Also available on the right panel.

<code>
{
  user_account_id: "ea39b907-6953-4354-abe4-99c441e5b1b3",
  baby_name: "Noah",
  bday: "2022-12-02",
  days: 250,
  sex: "MALE",
  weight_value: 3.6,
  weight_perc: 70,
  height_value: 80,
  height_perc: 60,
  bmi_value: 2.5,
  bmi_perc: 70,
  cooking_level: "EASY",
  goal: "Get inspiration",
  for_who: "FAMILY",
  adventure: "Y",
  repeatMenu: "Y",
  source: "SOCIAL_MEDIA",
  tags: [
    {user_account_id: "ea39b907-6953-4354-abe4-99c441e5b1b3", tag_id: "STAGE_FAMILY"},
    {user_account_id: "ea39b907-6953-4354-abe4-99c441e5b1b3", tag_id: "EQUIP_BLENDER"},
    {user_account_id: "ea39b907-6953-4354-abe4-99c441e5b1b3", tag_id: "GOALS_VEGGIES"}
  ],
  diet: [
    {user_account_id: "ea39b907-6953-4354-abe4-99c441e5b1b3", diet_type: "VEGAN"}
  ],
  allergy: [
    {user_account_id: "ea39b907-6953-4354-abe4-99c441e5b1b3", diet_type: "DAIRY"}
  ]
}
</code>

> Example input parameters

```graphql
{
  user_account_id: "ea39b907-6953-4354-abe4-99c441e5b1b3",
  baby_name: "Noah",
  bday: "2022-12-02",
  days: 250,
  sex: "MALE",
  weight_value: 3.6,
  weight_perc: 70,
  height_value: 80,
  height_perc: 60,
  bmi_value: 2.5,
  bmi_perc: 70,
  cooking_level: "EASY",
  goal: "Get inspiration",
  for_who: "FAMILY",
  adventure: "Y",
  repeatMenu: "Y",
  source: "SOCIAL_MEDIA",
  tags: [
    {user_account_id: "ea39b907-6953-4354-abe4-99c441e5b1b3", tag_id: "STAGE_FAMILY"},
    {user_account_id: "ea39b907-6953-4354-abe4-99c441e5b1b3", tag_id: "EQUIP_BLENDER"},
    {user_account_id: "ea39b907-6953-4354-abe4-99c441e5b1b3", tag_id: "GOALS_VEGGIES"}
  ],
  diet: [
    {user_account_id: "ea39b907-6953-4354-abe4-99c441e5b1b3", diet_type: "VEGAN"}
  ],
  allergy: [
    {user_account_id: "ea39b907-6953-4354-abe4-99c441e5b1b3", diet_type: "DAIRY"}
  ]
}
```

## Update Account Profile

```graphql
mutation UpdateAccount(
  $user_account_id: uuid!
  $nickname: String
  $bio: String
  $link: String
  $thumbnailLink: String!
) {
  update_user_account(
    where: {id: {_eq: $user_account_id}}
    _set: {nickname: $nickname, bio: $bio, thumbnail_link: $thumbnailLink, link: $link}
  ) {
    returning {
      id
    }
  }
}
```

TODO. Diet restriction and food allergy should be updated here in the same query in future, but this has not been implemented.


### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
user_account_id | uuid      | the user's database-generated ID
nickname      | text        | new nickname
bio           | text        | new bio
link          | text        | new bio link
thumbnailLink | text        | link to new thumbnail



## Update Baby Profile

```graphql
mutation UpdateBaby(
  $baby_id: uuid!
  $name: String!
  $sex_type: sex_type_enum
  $birthday: String
) {
  update_baby_profile(
    where: {id: {_eq: $baby_id}}
    _set: {name: $name, sex_type_id: $sex_type, birthday: $birthday}
  ) {
    returning {
      id
    }
  }
}
```


> Example input parameter

```graphql
{
  baby_id: "ea39b907-6953-4354-abe4-99c441e5b1b3",
  name: "New Name",
  sex_type: "MALE",
  birthday: "2024-11-03"
}
```

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
baby_id       | uuid        | the baby's database-generated ID
name          | text        | baby's new name
sex_type      | enum        | baby's updated gender; either 'MALE' or 'FEMALE'
birthday      | text        | baby's birthday in text format

### Example input parameter
<code>
{
  baby_id: "ea39b907-6953-4354-abe4-99c441e5b1b3",
  name: "New Name",
  sex_type: "MALE",
  birthday: "2024-11-03"
}
</code>



# Recipes

Make sure to have these filters:

- removed_at is null
- recipe_status_id == COMPLETE
- user_account.user_status is either ACTIVE_FREE or ACTIVE_PREMIUM


## Get all recipes

```graphql
query GetAllSlugs{
  recipe(where: 
    {_and: {
      recipe_status_id: {_eq: COMPLETE}, 
      user_account: {user_status: {_nin: [DEACTIVATED, SUSPENDED]}}}, 
      slug: {_is_null: false}
    }
  ) {
    user_account {
        nickname
    }
    slug
    updated_at
  }
}
```

This query is called only when web version is generating sitemal.xml.



## Search recipes by search term

```graphql
query SearchRecipe($search_term: String!) {
  recipe(where: {_and: [
    {removed_at: {_is_null: true}},
    {recipe_status_id: {_eq: COMPLETE}}, 
    {user_account: {user_status: {_in: [ACTIVE_FREE, ACTIVE_PREMIUM]}}}
    {_or: [
      {title: {_ilike: $search_term}}, 
      {description: {_ilike: $search_term}}, 
      {user_account: {nickname: {_ilike: $search_term}}},
      {recipe_ings: {display_name: {_ilike: $search_term}}},
      {recipe_tags: {tag: {value: {_ilike: $search_term}}}}
      ]
    }]}
  ) {
    id
    link_pic
    title
    slug
    user_account {
      nickname
    }
    base_serving_num
  }
}
```

Searching recipes based on title, description, recipe author's nickname, ingredients and tags.

<aside class="notice">
We track what terms are being searched. Make sure to save search using the second query
</aside>


### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
search_term   | text        | search term

```graphql
mutation SaveSearch($user_account_id: uuid!, $search_term: String!) {
  insert_search_record(objects: {user_account_id: $user_account_id, search_term: $search_term}){
    affected_rows
  }
}
```

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
user_account_id | uuid      | The user's database-generated ID
search_term   | text        | search term



## Fetch recipes for a specific ingredient

```graphql
query GetRecipeIngIntro($ing_intro: String!) {
  recipe(limit: 20, where: {_and: [
    {removed_at: {_is_null: true}},
    {recipe_status_id: {_eq: COMPLETE}}, 
    {user_account: {user_status: {_in: [ACTIVE_FREE, ACTIVE_PREMIUM]}}},
    {recipe_ings: {
      _or: [
        {ingredient: {ingredient_name: {_ilike: $ing_intro}}},
        {ingredient:{ing_intro: {name: {_ilike: $ing_intro}}}}
      ]}}]}
  ) {
    id
    link_pic
    title
    slug
    user_account {
      nickname
    }
  }
}

```

This functionality is used primarily for food tracking page. 


### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
ing_intro     | uuid        | Database-generated ID from ing_intro table




## Fetch recipes by one tag

```graphql
query GetRecipeByTag($tag: tag_enum!) {
  recipe(limit: 20, where: {_and: 
    [
      {recipe_tags: {tag_id: {_eq: $tag}}}, 
      {removed_at: {_is_null: true}},
      {recipe_status_id: {_eq: COMPLETE}}, 
      {user_account: {user_status: {_in: [ACTIVE_FREE, ACTIVE_PREMIUM]}}},
    ]}
  ) {
    id
    user_account {
      uid
      nickname
      thumbnail_link
    }
    title
    description
    link_pic
    base_serving_num
    slug
    recipe_tags {
      tag_id
    }
  }
}
```

This functionality can be used when there is one specific recipe tag that you want to fetch recipes for. On the main page, when a user picks one category among popular recipe categories, this is the query to use.



## Fetch recipes by more than one tags

```graphql
query GetRecipeByTag($tag: tag_enum!) {
  recipe(limit: 20, where: {_and: 
    [
      {recipe_tags: {tag_id: {_in: $tag}}}, 
      {removed_at: {_is_null: true}},
      {recipe_status_id: {_eq: COMPLETE}}, 
      {user_account: {user_status: {_in: [ACTIVE_FREE, ACTIVE_PREMIUM]}}},
    ]}
  ) {
    id
    user_account {
      uid
      nickname
      thumbnail_link
    }
    title
    description
    link_pic
    base_serving_num
    slug
    recipe_tags {
      tag_id
    }
  }
}
```

This functionality can be used when there is one or more specific recipe tags that you want to fetch. The main feed page on mobile app uses this query.


### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
tag           | text[]      | List of tag IDs

### Example input parameter

<code>
{
  tag: ["STAGE_ONE", "MORE_PROTEIN"]
}
</code>

> Example input parameter

```graphql
{
  tag: ["STAGE_ONE", "MORE_PROTEIN"]
}
```



## Fetch recipe detail by slug

```graphql
query GetRecipeDetail($slug: String!, $user_account_id: uuid!) {
  recipe(where: {slug: {_eq: $slug}}) {
    id
    user_account {
      uid
      nickname
      thumbnail_link
      id
    }
    title
    description
    link_pic
    base_serving_num
    slug
    recipe_tags {
      tag_id
    }
    recipe_nutritions(limit: 1) {
      added_sugar_G
      calcium_MG
      calories_KCAL
      carbohydrate_G
      cholesterol_MG
      choline_MG
      folatedfe_UG
      iron_MG
      magnesium_MG
      niacin_MG
      omega3_G
      omega6_G
      phosphorus_MG
      potassium_MG
      protein_G
      riboflavin_MG
      saturated_fatty_acids_G
      sodium_MG
      thiamin_MG
      total_dietary_fiber_G
      total_fatty_acids_G
      total_lipid_G
      total_sugar_G
      trans_fatty_acids_G
      vitamina_UG
      vitaminb12_UG
      vitaminb6_MG
      vitaminc_MG
      vitamind_UG
      vitamine_MG
      vitamink_UG
      zinc_MG
    }
    total_time_min
    prep_time_min
    cooking_time_min
    complexity_type_id
    recipe_ings {
      display_name
      display_amount
      ingredient{
        id
        food_group_id
        ing_intro{
          id
          health_tag
          is_allergen
        }
      }
    }
    recipe_steps {
      step_num
      description
    }
    comments {
      comment
      user_account {
        id
        nickname
        thumbnail_link
      }
    }
    likes_aggregate {
      aggregate {
        count(distinct: true, columns: liker_id)
      }
    }
    likes(where: {removed_at: {_is_null: true}, liker_id: {_eq: $user_account_id}}) {
      user_account {
        uid
      }
    }
  }
}
```
Every recipe has a slug. This query fetches recipe detail given the slug.

<aside class="notice">
This requires the user's token because it retrieves whether this user liked / commented on this recipe.
</aside>

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
slug          | text        | recipe slug
user_account_id | uuid      | the user's database-generated ID



## Fetch recipe detail by recipe ID

```graphql
query GetRecipeDetail($recipe_id: uuid!, $user_account_id: uuid!) {
  recipe(where: {id: {_eq: $recipe_id}}) {
    id
    user_account {
      uid
      nickname
      thumbnail_link
      id
    }
    title
    description
    link_pic
    base_serving_num
    slug
    recipe_tags {
      tag_id
    }
    recipe_nutritions(limit: 1) {
      added_sugar_G
      calcium_MG
      calories_KCAL
      carbohydrate_G
      cholesterol_MG
      choline_MG
      folatedfe_UG
      iron_MG
      magnesium_MG
      niacin_MG
      omega3_G
      omega6_G
      phosphorus_MG
      potassium_MG
      protein_G
      riboflavin_MG
      saturated_fatty_acids_G
      sodium_MG
      thiamin_MG
      total_dietary_fiber_G
      total_fatty_acids_G
      total_lipid_G
      total_sugar_G
      trans_fatty_acids_G
      vitamina_UG
      vitaminb12_UG
      vitaminb6_MG
      vitaminc_MG
      vitamind_UG
      vitamine_MG
      vitamink_UG
      zinc_MG
    }
    total_time_min
    prep_time_min
    cooking_time_min
    complexity_type_id
    recipe_ings {
      id
      display_name
      display_amount
      affiliate_link
      ingredient{
        id
        food_group_id
        ing_intro{
          id
          health_tag
          is_allergen
        }
      }
    }
    recipe_steps {
      step_num
      description
    }
    comments {
      comment
      user_account {
        id
        nickname
        thumbnail_link
      }
    }
    likes_aggregate {
      aggregate {
        count(distinct: true, columns: liker_id)
      }
    }
    likes(where: {removed_at: {_is_null: true}, liker_id: {_eq: $user_account_id}}) {
      user_account {
        uid
      }
    }
  }
}

```

<aside class="warning">
This query should retire. Please let Kay know when you see this being used.
</aside>


## Save favorite recipe

```graphql
mutation SaveFavoriteRecipe($user_account_id: uuid!, $recipe_id: uuid!) {
  insert_user_saved_recipe_one(object: {user_account_id: $user_account_id, recipe_id: $recipe_id}){
    id
  }
}
```

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
user_account_id | uuid      | the user's database-generated ID
recipe_id     | uuid        | recipe ID



## Create a new recipe

```graphql
mutation InsertRecipe(
  $author_id: uuid!, 
  $base_serving_num: Float!,
  $complexity_type_id: complexity_type_enum = MEDIUM, 
  $title: String!,
  $description: String = "",
  $link_pic: String = "", 
  $total_time_min: Float!,
  $slug: String!,
  $recipe_ings: [recipe_ing_insert_input!] = {},
  $recipe_nutrition: [recipe_nutrition_insert_input!] = {}, 
  $recipe_steps: [recipe_step_insert_input!] = {}, 
  $recipe_tags: [recipe_tag_insert_input!] = {} ) {
  insert_recipe(objects: {
    author_id: $author_id, 
    base_serving_num: $base_serving_num, 
    complexity_type_id: $complexity_type_id, 
    description: $description, 
    title: $title, 
    total_time_min: $total_time_min,
    link_pic: $link_pic, 
    slug: $slug,
    recipe_status_id: IN_REVIEW, 
    recipe_ings: {data: $recipe_ings}, 
    recipe_nutritions: {data: $recipe_nutrition}, 
    recipe_steps: {data: $recipe_steps}, 
    recipe_tags: {data: $recipe_tags}, 
    }){
      returning{
        id
      }
    }
}
```

Currently this is allowed in the mobile app, but we should get this built out on web shortly.

```graphql
{
  "author_id": "ea39b907-6953-4354-abe4-99c441e5b1b3",
  "base_serving_num": 1.5,
  "complexity_type_id": "EASY",
  "title": "Title test",
  "description": "Description",
  "link_pic": "",
  "total_time_min": 15,
  "recipe_ings": [{
    "ing_id": "28b39ab0-ca7b-4305-8c84-010737a1fce7",
    "display_amount": 1,
    "display_unit_id": "CUP",
    "std_amount": 15,
    "std_unit_id": "GRAM_RAW",
    "display_name": "Spinach"
  }, {
    "ing_id": "1776a20d-beac-41b2-a4b4-f6353f0a775e",
    "display_amount": 1,
    "display_unit_id": "CUP",
    "std_amount": 15,
    "std_unit_id": "GRAM_RAW",
    "display_name": "Carrot, cubed"
  }],
  "recipe_nutrition": [{
    "added_sugar_G": 0,
    "calcium_MG": 1,
    "calories_KCAL": 2,
    "carbohydrate_G": 3,
    "cholesterol_MG": 4,
    "choline_MG": 5,
    "folatedfe_UG": 6,
    "iron_MG": 7,
    "magnesium_MG": 8,
    "niacin_MG": 9,
    "omega3_G": 10,
    "omega6_G": 11,
    "phosphorus_MG": 12,
    "potassium_MG": 13,
    "protein_G": 14,
    "riboflavin_MG": 15,
    "saturated_fatty_acids_G": 16,
    "sodium_MG": 17,
    "thiamin_MG": 18,
    "total_dietary_fiber_G": 19,
    "total_lipid_G": 20,
    "vitamina_IU": 21,
    "vitaminb12_UG": 22,
    "vitaminb6_MG": 23,
    "vitaminc_MG": 24,
    "vitamind_UG": 25,
    "vitamine_MG": 26,
    "vitamink_UG": 27,
    "zinc_MG": 28,
    "total_sugar_G": 29,
    "total_fatty_acids_G": 30,
    "trans_fatty_acids_G": 31
  }],
  "recipe_steps": [{
    "step_num": 1,
    "description": "description1"
  }, {
    "step_num": 2,
    "description": "description2"
  }],
  "recipe_tags": [{
    "tag_id": "STAGE_ONE",
    "is_primary": false
  }]
}
```



## Edit a recipe
Currently this is allowed in the mobile app, but we should have this built out on web shortly.

```graphql

  mutation EditRecipe(
    $recipe_id: uuid!, 
    $base_serving_num: Float = 1, 
    $complexity_type_id: complexity_type_enum = COMPLEX, 
    $description: String = "", 
    $title: String = "", 
    $slug: String = "",
    $total_time_min: Float = 180, 
    $recipe_nutrition: recipe_nutrition_set_input = {},
    $recipe_tags: [recipe_tag_insert_input!] = {}, 
    $recipe_steps: [recipe_step_insert_input!] = {}, 
    $recipe_ings: [recipe_ing_insert_input!] = {}) {

    update_recipe(where: {id: {_eq: $recipe_id}}, 
      _set: {
        base_serving_num: $base_serving_num, 
        complexity_type_id: $complexity_type_id, 
        description: $description, 
        title: $title, 
        total_time_min: $total_time_min,
        slug: $slug
      }) {
      returning {
        id
      }
    }
    update_recipe_nutrition(where: {recipe_id: {_eq: $recipe_id}}, _set: $recipe_nutrition){
      returning {
        id
      }
    }
    delete_recipe_ing(where: {recipe_id: {_eq: $recipe_id}}) {
      returning {
        id
      }
    }
    insert_recipe_ing(objects: $recipe_ings) {
      returning {
        id
      }
    }

    delete_recipe_step(where: {recipe_id: {_eq: $recipe_id}}) {
      returning {
        id
      }
    }
    insert_recipe_step(objects: $recipe_steps) {
      returning {
        id
      }
    }
    delete_recipe_tag(where: {recipe_id: {_eq: $recipe_id}}){
      returning {
        id
      }
    }
    insert_recipe_tag(objects: $recipe_tags){
      returning {
        id
      }
    }
  }

```

```graphql
{
  "recipe_id": "065d689d-2b52-4cf1-86b1-a221380e1b28",
  "base_serving_num": 1.5,
  "complexity_type_id": "EASY",
  "title": "Title test",
  "description": "Description",
  "total_time_min": 15,
  "recipe_ings": [{
    "recipe_id": "065d689d-2b52-4cf1-86b1-a221380e1b28",
    "ingredient_id": "4fbc6b95-2bd0-4655-9c30-112d9521980b",
    "display_amount": "1 CUP",
    "std_amount": 15,
    "display_name": "Spinach"
  }, {
    "recipe_id": "065d689d-2b52-4cf1-86b1-a221380e1b28",
    "ingredient_id": "4fbc6b95-2bd0-4655-9c30-112d9521980b",
    "display_amount": "1 CUP",
    "std_amount": 15,
    "display_name": "Carrot, cubed"
  }],
  "recipe_nutrition": {
    "added_sugar_G": 0,
    "calcium_MG": 1,
    "calories_KCAL": 2,
    "carbohydrate_G": 3,
    "cholesterol_MG": 4,
    "choline_MG": 5,
    "folatedfe_UG": 6,
    "iron_MG": 7,
    "magnesium_MG": 8,
    "niacin_MG": 9,
    "omega3_G": 10,
    "omega6_G": 11,
    "phosphorus_MG": 12,
    "potassium_MG": 13,
    "protein_G": 14,
    "riboflavin_MG": 15,
    "saturated_fatty_acids_G": 16,
    "sodium_MG": 17,
    "thiamin_MG": 18,
    "total_dietary_fiber_G": 19,
    "total_lipid_G": 20,
    "vitamina_UG": 21,
    "vitaminb12_UG": 22,
    "vitaminb6_MG": 23,
    "vitaminc_MG": 24,
    "vitamind_UG": 25,
    "vitamine_MG": 26,
    "vitamink_UG": 27,
    "zinc_MG": 28,
    "total_sugar_G": 29,
    "total_fatty_acids_G": 30,
    "trans_fatty_acids_G": 31
  },
  "recipe_steps": [{
    "recipe_id": "065d689d-2b52-4cf1-86b1-a221380e1b28",
    "step_num": 1,
    "description": "description1"
  }, {
    "recipe_id": "065d689d-2b52-4cf1-86b1-a221380e1b28",
    "step_num": 2,
    "description": "description2"
  }],
  "recipe_tags": [{
    "recipe_id": "065d689d-2b52-4cf1-86b1-a221380e1b28",
    "tag_id": "STAGE_ONE"
  }]
}
```

## Delete a recipe

```graphql
mutation DeleteRecipe($recipe_id: uuid!) {
  update_recipe(where: {id: {_eq: $recipe_id}}, _set: {removed_at: now, recipe_status_id: ARCHIVE}){
    returning {
      id
    }
  }
}
```

# Meal Plans

## Save a recipe to meal plan

```graphql
mutation AddToMealPlan(
  $planned_at: timestamp!, 
  $recipe_id: uuid!, 
  $user_account_id: uuid!
) {
  insert_recipe_hist(objects: {
    user_account_id: $user_account_id, 
    recipe_id: $recipe_id, 
    planned_at: $planned_at}
  ){
    returning {
      id
    }
  }
}
```

> Example input parameter

```graphql
{
  user_account_id: "065d689d-2b52-4cf1-86b1-a221380e1b28",
  recipe_id: "065d689d-2b52-4cf1-86b1-a221380e1b28",
  planned_at: "2024-11-02"
}
```

This is used when a user manually adds a recipe to a meal plan

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
user_account_id | uuid      | the user's database-generated ID
recipe_id     | uuid        | recipe ID
planned_at    | timestamp   | the date that a recipe is added to in meal plan

### Example input parameters

<code>
{
  user_account_id: "065d689d-2b52-4cf1-86b1-a221380e1b28",
  recipe_id: "065d689d-2b52-4cf1-86b1-a221380e1b28",
  planned_at: "2024-11-02"
}
</code>


## Save a batch of meal plan

```graphql
mutation AddAllFromRecEngine($objects: [recipe_hist_insert_input!] = {}) {
  insert_recipe_hist(objects: $objects){
    returning{
      id
      recipe {
        recipe_nutritions {
          added_sugar_G
          calcium_MG
          calories_KCAL
          carbohydrate_G
          cholesterol_MG
          choline_MG
          folatedfe_UG
          iron_MG
          magnesium_MG
          niacin_MG
          omega3_G
          omega6_G
          phosphorus_MG
          potassium_MG
          protein_G
          riboflavin_MG
          saturated_fatty_acids_G
          sodium_MG
          thiamin_MG
          total_dietary_fiber_G
          total_fatty_acids_G
          total_lipid_G
          total_sugar_G
          trans_fatty_acids_G
          vitamina_UG
          vitaminb12_UG
          vitaminb6_MG
          vitaminc_MG
          vitamine_MG
          vitamind_UG
          vitamink_UG
          zinc_MG
        }
      }
    }
  }
}
```

```graphql
{
  "objects": [
    {"recipe_id": "6a990e5b-d42c-4d10-9794-fe42ca888b19",
     "planned_at": "1/8/2024",
     "user_account_id": "4782a261-7d65-4b2a-8850-a4505320d3fc"},
    {"recipe_id": "6a990e5b-d42c-4d10-9794-fe42ca888b19",
     "planned_at": "1/9/2024",
     "user_account_id": "4782a261-7d65-4b2a-8850-a4505320d3fc"}
  ]
}
```

The query above returns recipe_hist_id and recipe.recipe_nutrition.



## Remove from meal plan
```graphql
mutation RemoveMealPlan($recipe_hist_id: uuid!) {
  update_recipe_hist(where: {id: {_eq: $recipe_hist_id}}, _set: {removed_at: now}){
    returning{
      id
    }
  }
}
```

<aside class="notice">
Note that it is purely going by recipe_hist_id. The user-specificity is enforced in jwt token.
</aside>


## Update one meal plan menu

```graphql
mutation UpdateMealPlan($recipe_hist_id: uuid!, $planned_at: timestamp!) {
  update_recipe_hist(where: {id: {_eq: $recipe_hist_id}}, _set: {planned_at: $planned_at}){
    returning{
      id
    }
  }
}
```

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
recipe_hist_id | uuid       | the user's meal plan data entry ID from recipe_hist table
planned_at    | timestamp   | the date that a recipe is added to in meal plan



## Read meal plan

```graphql
query GetMealPlan(
  $start_at: timestamp!, 
  $user_account_id: uuid!
) {
  recipe_hist(where: {
    removed_at: {_is_null: true}, 
    planned_at: {_gte: $start_at}, 
    user_account_id: {_eq: $user_account_id}
  }) {
    id
    planned_at
    served_at
    reaction_id
    served_amount
    note
    leftover
    recipe {
      id
      title
      link_pic
      user_account {
        nickname
      }
    }
  }
}

```

> Example input parameter:

```graphql
{
  user_account_id: "a1c5a247-b70d-4c0c-9f84-9f6bd5243391",
  start_at: "2024-11-01"
}
```


### Parameters:

Parameter | Type        | Description
--------- | ----------- | -----------
user_account_id | uuid  | the user's database-generated ID
start_at  | timestamp   | data load look back date. E.g. if entered '2024-11-01', it fetches meal plan from '2024-11-01'.


## Save tracking for meal plan

```graphql
mutation SaveReaction(
    $recipe_hist_id: uuid!, 
    $reaction_id: reaction_type_enum!, 
    $served_amount: Float!,
    $note: String="",
    $leftover: String="",
) {
  update_recipe_hist(
    where: {id: {_eq: $recipe_hist_id}}, 
    _set: {
      reaction_id: $reaction_id, 
      served_amount: $served_amount,
      served_at:now,
      note:$note,
      leftover: $leftover
    }
  ) {
    returning {
      served_amount
      leftover
      recipe {
        recipe_ings {
          ingredient {
            ing_intro {
              id
              allergen_type
              name
            }
            food_group_id
          }
        }
      }
    }
  }
}
```


> Example input parameter:

```graphql
{
  recipe_hist_id: "a1c5a247-b70d-4c0c-9f84-9f6bd5243391",
  reaction_id: "LIKE",
  served_amount: 1.5,
  note: "It was good",
  leftover: "Didn't eat broccoli"
}
```


### Parameters:

Parameter         | Type        | Description
----------------- | ----------- | -----------
reaction_hist_id  | uuid        | meal plan entry ID from recipe_hist_id
reaction_id       | enum        | LIKE, NEUTRAL, or DISLIKE
served_amount     | real        | the serving size that the kid ate
note              | text        | note about this intake
leftover          | text        | description of what was being left


### Example input parameter
<code>
{
  recipe_hist_id: "a1c5a247-b70d-4c0c-9f84-9f6bd5243391",
  reaction_id: "LIKE",
  served_amount: 1.5,
  note: "It was good",
  leftover: "Didn't eat broccoli"
}
</code>


## Save milestones

```graphql
mutation AddMilestone($objects: [user_milestone_insert_input!]!) {
  insert_user_milestone(
    objects: $objects,
    on_conflict: {
      constraint: user_milestone_pkey,    
      update_columns: [] 
    }
  ) {
    returning {
      id
    }
  }
}
```

> Example input parameter

```graphql
{
  "objects": [
    {
      "user_account_id": "a1c5a247-b70d-4c0c-9f84-9f6bd5243391",
    	"milestone_id": "ALLERGEN_SHELLFISH" 
    },
    {
      "user_account_id": "a1c5a247-b70d-4c0c-9f84-9f6bd5243391",
    	"milestone_id": "ALLERGEN_EGG" 
    }
  ]
}
```


### Parameters:

Parameter         | Type        | Description
----------------- | ----------- | -----------
objects           | list        | list of {user_account_id, milestone_id}

### Example input parameter

<code>
{
  "objects": [
    {
      "user_account_id": "a1c5a247-b70d-4c0c-9f84-9f6bd5243391",
    	"milestone_id": "ALLERGEN_SHELLFISH" 
    },
    {
      "user_account_id": "a1c5a247-b70d-4c0c-9f84-9f6bd5243391",
    	"milestone_id": "ALLERGEN_EGG" 
    }
  ]
}
</code>


# Social


## Get feed

```graphql
query GetFeed(
  $recipe_offset: Int!, 
  $recipe_limit: Int!, 
  $user_account_id: uuid!
) @cached {
  recipe(
    offset: $recipe_offset, 
    limit: $recipe_limit, 
    order_by: {created_at: desc},
    where: {
      recipe_status_id: {_eq: COMPLETE}, 
      user_account: {_or: [
        {uid: {_ilike: "%admin%"}},
        {id: {_eq: $user_account_id}},
        {user_follows: {follower_id: {_eq: $user_account_id}}}]}}
  ) {
    id
    user_account {
      uid
      nickname
      thumbnail_link
    }
    link_pic
    title
    description
    created_at
    updated_at
    comments(where: {removed_at: {_is_null: true}}) {
      user_account {
        id
        nickname
        thumbnail_link
      }
      comment
    }
    likes_aggregate {
      aggregate {
        count(distinct: true, columns: liker_id)
      }
    }
    likes(where: {removed_at: {_is_null: true}, user_account: {id: {_eq: $user_account_id}}}) {
      user_account {
        uid
      }
    }
  }
  user_account(
    where: {
      _and: [
        { id: { _neq: $user_account_id } },              
        {_not: {user_follows: { follower_id: { _eq: $user_account_id }}}}
      ]
    },
    order_by: { recipes_aggregate: { count: desc } }
    limit: 5
  ) {
    id
    nickname
    thumbnail_link
  }
}
```

> Example input parameter

```graphql
{
    "user_account_id": "7177a1bb-875f-4836-939a-7600e7c134cd"
    "recipe_offset": 0,
    "recipe_limit": 10,
}
```

Note that this query outputs five other users that we recommend to follow. However, currently we do not properly filter out those that the user is already following because of limitation in query method. We will revisit this.

TODO: <code>{user_follows: {follower_id: {_neq: $user_account_id}}}</code> put this back to filter out the accounts that user is already following.


### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
user_account_id | uuid      | the user's database-generated ID
recipe_offset | integer     | the number of recipes as offset
recipe_limit  | integer     | limit in the number of recipes to return

### Example input parameters

<code>
{
    "user_account_id": "7177a1bb-875f-4836-939a-7600e7c134cd"
    "recipe_offset": 0,
    "recipe_limit": 10,
}
</code>


## Fetch a user profile
```graphql
query fetchUserData($nickname: String!) {
  user_account(where: {nickname: {_eq: $nickname}}) {
    id
    nickname
    bio
    link
    user_type
    thumbnail_link
    user_follows(where: {removed_at: {_is_null: true}}, distinct_on: follower_id) {
      follower_id
    }
    userFollowsByFollowerId(where: {removed_at: {_is_null: true}}, distinct_on: followee_id) {
      followee_id
    }
    recipes(where: {_and: {
      recipe_status_id: {_eq: COMPLETE}, 
      user_account: {user_status: {_nin: [DEACTIVATED, SUSPENDED]}}}, 
      slug: {_is_null: false}}
    ){
      id
      title
      link_pic
    }
    user_saved_recipes{
      recipe{
        id
        title
        link_pic
      }
    }
  }
}
```

> Example input parameter

```graphql
{
  nickname: "heartful-blw"
}
```

This is used when one visits a user's page. This fetches data to populate the visited user's page.

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
nickname      | text        | nickname of the visitee

### Example input parameters

<code>
{
  nickname: "heartful-blw"
}
</code>

## Comment on a recipe

```graphql
mutation InsertComment($commenter_id: uuid!, $recipe_id: uuid!, $comment: String!) {
  insert_comments_one(object: {commenter_id: $commenter_id, recipe_id: $recipe_id, comment: $comment}) {
    id
  }
}
```

> Example input parameter

```graphql
{
  commenter_id: "065d689d-2b52-4cf1-86b1-a221380e1b28",
  recipe_id: "065d689d-2b52-4cf1-86b1-a221380e1b28",
  comment: "Looking delicious!"
}
```

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
user_account_id | uuid      | the user's database-generated ID
recipe_id     | uuid        | recipe ID
comment       | text        | comment

### Example input parameters

<code>
{
  commenter_id: "065d689d-2b52-4cf1-86b1-a221380e1b28",
  recipe_id: "065d689d-2b52-4cf1-86b1-a221380e1b28",
  comment: "Looking delicious!"
}
</code>


## Like a recipe

```graphql
mutation InsertLike($liker_id: uuid!, $recipe_id: uuid!) {
  insert_likes_one(object: {liker_id: $liker_id, recipe_id: $recipe_id}){
    id
  }
}
```

> Example input parameter

```graphql
{
  "liker_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
  "recipe_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
}
```

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
liker_id      | uuid        | the user's database-generated ID
recipe_id     | uuid        | recipe ID

### Example input parameters

<code>
{
  "liker_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
  "recipe_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
}
</code>



## Unlike a recipe

```graphql
mutation RemoveLike ($liker_id: uuid!, $recipe_id:uuid!){
  update_likes(where: {liker_id: {_eq: $liker_id}, recipe_id: {_eq: $recipe_id}}, 
    _set: {removed_at: now}){
    returning {
      id
    }
  }
}
```

> Example input parameter

```graphql
{
  "liker_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
  "recipe_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
}
```

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
liker_id      | uuid        | the user's database-generated ID
recipe_id     | uuid        | recipe ID

### Example input parameters

<code>
{
  "liker_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
  "recipe_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
}
</code>



## Follow a user

```graphql
mutation InsertFollow ($follower_id: uuid!, $followee_id: uuid!) {
  insert_user_follow_one(object: {follower_id: $follower_id, followee_id: $followee_id}) {
    id
  }
}
```

> Example input parameter

```graphql
{
  "follower_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
  "followee_id": "7177a1bb-875f-4836-939a-7600e7c134cd"
}
```

Only a follower can insert a relationship here. 

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
follower_id   | uuid        | the user ID of the follower
followee_id   | uuid        | the user ID of the followee

### Example input parameters

<code>
{
  "follower_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
  "followee_id": "7177a1bb-875f-4836-939a-7600e7c134cd"
}
</code>




## Unfollow a user

```graphql
mutation UpdateUnfollow($follower_id: uuid!, $followee_id: uuid!) {
  update_user_follow(where: {followee_id: {_eq: $followee_id}, follower_id: {_eq: $follower_id}}, _set: {removed_at: now}){
    returning {
      id
    }
  }
}
```

> Example input parameter

```graphql
{
  "follower_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
  "followee_id": "7177a1bb-875f-4836-939a-7600e7c134cd"
}
```

Only a follower can delete a relationship here. 

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
follower_id   | uuid        | the user ID of the follower
followee_id   | uuid        | the user ID of the followee

### Example input parameters

<code>
{
  "follower_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
  "followee_id": "7177a1bb-875f-4836-939a-7600e7c134cd"
}
</code>


## Report a recipe
```graphql
mutation ReportRecipe($reporter_id: uuid = "", $recipe_id: uuid = "") {
  insert_flag(objects: {reporter_id: $reporter_id, recipe_id: $recipe_id}) {
    returning{
    reporter_id      
    }
  }
}
```

> Example input parameter

```graphql
{
  "reporter_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
  "recipe_id": "7177a1bb-875f-4836-939a-7600e7c134cd"
}
```

Only a follower can delete a relationship here. 

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
reporter_id   | uuid        | the user ID
recipe_id     | uuid        | the recipe ID 

### Example input parameters

<code>
{
  "reporter_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
  "recipe_id": "7177a1bb-875f-4836-939a-7600e7c134cd"
}
</code>


## Report a user
```graphql
mutation ReportUser($reporter_id: uuid = "", $user_id: uuid = "") {
  insert_flag(objects: {reporter_id: $reporter_id, user_id: $user_id}) {
    returning{
    reporter_id      
    }
  }
}
```

> Example input parameter

```graphql
{
  "reporter_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
  "user_id": "7177a1bb-875f-4836-939a-7600e7c134cd"
}
```

Only a follower can delete a relationship here. 

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
reporter_id   | uuid        | ID of the user that is being flagged
uer_id        | uuid        | ID of the user who is reporting

### Example input parameters

<code>
{
  "reporter_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
  "recipe_id": "7177a1bb-875f-4836-939a-7600e7c134cd"
}
</code>


# Health Records

Health records include growth data, and food reaction.

## Growth Records

### Create 

```graphql
mutation EnterGrowthData($growth: [baby_growth_hist_insert_input!] = {}) {
  insert_baby_growth_hist(objects: $growth) {
    returning {
      baby_id
    }
  }
}
```

> Example input parameters:

```graphql
const growthFormatted = [
  {
    baby_id: 'cd1783ea-7f56-4f42-8931-00638f801113',
    days: 247,
    growth_type_id: 'WEIGHT',
    percentile: 87,
    value: 6.5
  }
  {
    baby_id: 'cd1783ea-7f56-4f42-8931-00638f801113',
    days: 247,
    growth_type_id: 'HEIGHT',
    percentile: 87,
    value: 6.5
  }
  {
    baby_id: 'cd1783ea-7f56-4f42-8931-00638f801113',
    days: 247,
    growth_type_id: 'BMI',
    percentile: 87,
    value: 6.5
  }
]

```

> Example call in Apollo

```graphql
const [insertGrowth] = useMutation(GQL_ENTER_GROWTH);

insertGrowth({
  variables: { growth: growthFormatted }
})
```

Input data should be a list of objects with the following fields:

Parameter       | Type        | Description
--------------- | ----------- | -----------
baby_id         | uuid        | ID from baby_profile table
days            | number      | the baby's age in days
growth_type_id  | enum        | 'WEIGHT', 'HEIGHT', or 'BMI'
percentile      | integer     | the growth percentile that the app has calculated
value           | float       | the growth measure for weight, height, or bmi




### Update
Currently this functionality is not implemented. Users delete a record and re-enter the data.


### Delete

Currently this functionality deletes ALL growth data in the given ageDays for the baby.

```graphql
mutation DeleteGrowthData($baby_id: uuid!, $ageDays: Int!) {
  delete_baby_growth_hist(where: {_and: {baby_id: {_eq: $baby_id}, days: {_eq: $ageDays}}}){
    returning{
      id
    }
  }
}
```

Parameter | Type | Description
--------- | ----------- | -----------
baby_id | uuid | ID from baby_profile table
daysDays | number | the baby's age in days


## Food Reactions

### Create

```graphql
mutation EnterReactionData(
  $user_account_id: uuid!, 
  $entered_at: String!, 
  $baby_profile_id: uuid = null, 
  $reaction: [String!] = null, 
  $note: String = null) {
  insert_baby_reaction_hist(objects: {
    user_account_id: $user_account_id, 
    baby_profile_id: $baby_profile_id, 
    entered_at: $entered_at, 
    reaction: $reaction, 
    note: $note}){
      returning {
        id
      }
    }
}
```


> Example input parameter

```graphql
{
  user_account_id: "ea39b907-6953-4354-abe4-99c441e5b1b3",
  entered_at: "2024-11-03"
  baby_profile_id: "ea39b907-6953-4354-abe4-99c441e5b1b3",
  reaction: "HIVE",
  note: "Got hive..."
}
```

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
user_account_id | uuid      | database-generated ID of the user
baby_profile_id | uuid      | the baby's database-generated ID
entered_at    | timestamp   | the record time
reaction      | enum        | reaction types (see Codes)
note          | text        | description of the reaction

### Example input parameter
<code>
{
  user_account_id: "ea39b907-6953-4354-abe4-99c441e5b1b3",
  entered_at: "2024-11-03"
  baby_profile_id: "ea39b907-6953-4354-abe4-99c441e5b1b3",
  reaction: "HIVE",
  note: "Got hive..."
}
</code>


### Update

```graphql
mutation UpdateReaction(
  $id: uuid!, 
  $entered_at: String!
  $note: String = null, 
  $reaction: [String!] = null, 
) {
  update_baby_reaction_hist(
    where: {id: {_eq: $id}}, 
    _set: {note: $note, reaction: $reaction, entered_at: $entered_at}
    ){
      returning {
        id
      }
  }
}
```

> Example input parameter

```graphql
{
  id: "ea39b907-6953-4354-abe4-99c441e5b1b3",
  entered_at: "2024-11-03"
  reaction: "HIVE",
  note: "Got hive..."
}
```

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
id            | uuid        | ID of existing reaction entry
entered_at    | timestamp   | the record time
reaction      | enum        | reaction types (see Codes)
note          | text        | description of the reaction

### Example input parameter
<code>
{
  id: "ea39b907-6953-4354-abe4-99c441e5b1b3",
  entered_at: "2024-11-03"
  reaction: "HIVE",
  note: "Got hive..."
}
</code>


### Delete

```graphql
mutation DeleteReaction(
  $id: uuid!, 
) {
  update_baby_reaction_hist(
    where: {id: {_eq: $id}}, 
    _set: {removed_at: now}
  ){
    returning {
      id
    }
  }
}
```

> Example input parameter

```graphql
{
  id: "ea39b907-6953-4354-abe4-99c441e5b1b3"
}
```

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
id            | uuid        | ID of existing record entry

### Example input parameter
<code>
{
  id: "ea39b907-6953-4354-abe4-99c441e5b1b3"
}
</code>


# Food Tracking

Babies are supposed to get introduced to one new food at a time. Food tracking is to track which foods got introduced.


## Update food intro status

```graphql
mutation UpdateIngIntroStatus(
  $baby_id: uuid!, 
  $ing_intro_id: uuid!,
  $attempted_at: timestamp = null,
  $mastered_at: timestamp = null) {
  insert_baby_ing_intro_one(object: {
    baby_id: $baby_id, 
    ing_intro_id: $ing_intro_id, 
    attempted_at: $attempted_at, 
    mastered_at: $mastered_at
  }, on_conflict: {
    constraint: baby_ing_intro_baby_id_ing_intro_id_key,
    update_columns: [attempted_at, mastered_at]
  }){
    id
  }
}
```

> Example input parameter

```graphql
{
  baby_id: "ea39b907-6953-4354-abe4-99c441e5b1b3",
  ing_intro_id: "ea39b907-6953-4354-abe4-99c441e5b1b3",
  attempted_at: "2024-11-04",
  mastered_at: null
}
```

### Parameters

Parameter     | Type        | Description
------------- | ----------- | -----------
baby_id       | uuid        | baby ID
ing_intro_id  | uuid        | food item ID
attempted_at  | timestamp   | timestamp of first attempting this food
mastered_at   | timestamp   | timestamp of mastering this food intro


### Example input parameter
<code>
{
  baby_id: "ea39b907-6953-4354-abe4-99c441e5b1b3",
  ing_intro_id: "ea39b907-6953-4354-abe4-99c441e5b1b3",
  attempted_at: "2024-11-04",
  mastered_at: null
}
</code>


# Codes

## Recipe Tag
Value | Comment | Type
-------|---------|------
COURSE_APPETIZER | Appetizers | Course
COURSE_BEVERAGES | Beverages | Course
COURSE_BREADS | Breads | Course
COURSE_BREAKFAST | Breakfast | Course
COURSE_DESSERTS | Desserts | Course
COURSE_MAIN | Main dishes | Course
COURSE_PUREE | Purees | Course
COURSE_SALADS | Salads | Course
COURSE_SANDWICHES | Sandwiches | Course
COURSE_SAUCE | Dressings and sauces | Course
COURSE_SIDE | Sides | Course
COURSE_SNACK | Snack | Course
COURSE_SOUP | Soups and stews | Course
GOALS_VEGGIES | Vegetables and fruits | Nutrition Goal
GOALS_SEAFOOD | Seafoods | Nutrition Goal
GOALS_GRAINS | Whole grains | Nutrition Goal
GOALS_FAT_FREE | Fat free or low | Nutrition Goal
GOALS_PROTEIN | Vary protein options | Nutrition Goal
GOALS_CALCIUM | Calcium | Nutrition Goal
GOALS_LOW_SAT_FAT | Low saturated fat | Nutrition Goal
GOALS_LOW_SODIUM | Low sodium | Nutrition Goal
FOODGROUP_FRUITS | Fruits | Food Group
FOODGROUP_VEGETABLES | Vegetables | Food Group
FOODGROUP_GRAINS | Grains | Food Group
FOODGROUP_PROTEINS | Proteins | Food Group
FOODGROUP_DAIRY | Dairy | Food Group
EQUIP_BLENDER | Blender | Equipment
EQUIP_NOTHING | No equipment needed | Equipment
EQUIP_GRILL | Grill | Equipment
EQUIP_MICROWAVE | Microwave | Equipment
EQUIP_OVEN | Oven | Equipment
EQUIP_STOVETOP | Stovetop | Equipment
EQUIP_SLOW_COOKER | Slow cooker | Equipment
EQUIP_UTENSIL | Forks and spoons | Equipment
EQUIP_TOASTER | Toaster | Equipment
EQUIP_ICE_CUBE | Ice cube trays | Equipment
CUISINE_AMERICAN | American | Cuisine
CUISINE_ASIAN | Asian | Cuisine
CUISINE_HISPANIC | Hispanic | Cuisine
CUISINE_MEDITERRANEAN | Mediterranean | Cuisine
CUISINE_MIDDLE_EASTERN | Middle eastern | Cuisine
CUISINE_NATIVE_AMERICAN | Native American | Cuisine
CUISINE_SOUTHERN | Southern | Cuisine
CUISINE_VEGETARIAN | Vegetarian | Cuisine
CUISINE_VEGAN | Vegan | Cuisine
CUISINE_HALAL | Halal | Cuisine
CUISINE_KOSHER | Kosher | Cuisine
SEASONAL_EASTER | Easter | Seasonal
SEASONAL_THANKSGIVING | Thanksgiving | Seasonal
SEASONAL_HANUKKAH | Hanukkah | Seasonal
SEASONAL_CHRISTMAS | Christmas | Seasonal
SEASONAL_NEW_YEAR | New year | Seasonal
SEASONAL_LUNAR_NEW_YEAR | Lunar new year | Seasonal
SEASONAL_CINCO_DE_MAYO | Cinco de mayo | Seasonal
ALLERGEN_INTRO_MILK | Allergen Intro (Milk) | Allergen Intro
ALLERGEN_INTRO_EGG | Allergen Intro (Egg) | Allergen Intro
ALLERGEN_INTRO_PEANUTS | Allergen Intro (Peanuts) | Allergen Intro
ALLERGEN_INTRO_TREENUTS | Allergen Intro (Treenuts) | Allergen Intro
ALLERGEN_INTRO_SOY | Allergen Intro (Soy) | Allergen Intro
ALLERGEN_INTRO_WHEAT | Allergen Intro (Wheat) | Allergen Intro
ALLERGEN_INTRO_FISH | Allergen Intro (Fish) | Allergen Intro
ALLERGEN_INTRO_SHELLFISH | Allergen Intro (Shellfish) | Allergen Intro
ALLERGEN_FREE_MILK | Avoid allergen (Milk) | Allergen Free
ALLERGEN_FREE_EGG | Avoid allergen (Egg) | Allergen Free
ALLERGEN_FREE_PEANUTS | Avoid allergen (Peanuts) | Allergen Free
ALLERGEN_FREE_TREENUTS | Avoid allergen (Treenuts) | Allergen Free
ALLERGEN_FREE_SOY | Avoid allergen (Soy) | Allergen Free
ALLERGEN_FREE_WHEAT | Avoid allergen (Wheat) | Allergen Free
ALLERGEN_FREE_FISH | Avoid allergen (Fish) | Allergen Free
ALLERGEN_FREE_SHELLFISH | Avoid allergen (Shellfish) | Allergen Free
CUISINE_INDIAN | Indian | Cuisine
TEXTURE_SMOOTH | Smooth puree | Texture
TEXTURE_THICKER | Thicker Puree | Texture
TEXTURE_CHEWABLE | Soft and chewable | Texture
TEXTURE_WHOLE | Whole foods | Texture
STAGE_ONE | Stage 1 (4-6m) | Stage
STAGE_TWO | Stage 2 (6-9m) | Stage
STAGE_THREE | Stage 3 (9-12m) | Stage
STAGE_BLW | Finger foods (BLW) | Stage
STAGE_TODDLER | Toddler foods | Stage
STAGE_FAMILY | Family foods | Stage
SEASONAL | Seasonal | Category
HEALTH_GUT | Gut health | Health
HEALTH_IMMUNE | Immune system | Health
HEALTH_CARDIO | Cardiovascular health | Health
HEALTH_SKIN | Skin health | Health
HEALTH_BRAIN | Brain development | Health
HEALTH_VISION | Vision | Health
HEALTH_BONE | Bone health | Health
HEALTH_NERVE | Cognitive system | Health
HEALTH_SUPPORTIVE | Supporting other nutrition absorption | Health
HEALTH_ANTIOXIDANT | Anti-inflammatory, anti-oxidant, and anti-aging | Health
HEALTH_CELL | Cellular growth | Health
HEALTH_DIGESTION | Digestion | Health
HEALTH_ENERGY | Energy | Health
HEALTH_BLOOD | Blood health | Health
HEALTH_METABOLISM | Healthy metabolism | Health
GOALS_IRON | Iron sufficient | Nutrition Goal
EQUIP_AIRFRYER | Air fryer | Equipment
ALLERGEN_INTRO_SESAME | Introduce sesame | Allergen Intro
ALLERGEN_FREE_SESAME | Does not contain sesame | Allergen Free
OTHERS_COOKING_ACTIVITIES | Cooking with kids | Other

## Diet Restriction

Value | Comment
-------|--------
VEGETARIAN | Vegetarian
VEGAN | Vegan
PESCATARIAN | Pescatarian
HALAL | Halal
KOSHER | Kosher
GLUTEN_FREE | Gluten-free
FOOD_ALLERGY | Food Allergy
LACTOSE_INTOLERANT | Lactose-intolerant


## Food Allergy

| Value                     | Comment        | Category           |
|---------------------------|----------------|--------------------|
| COMMON_MILK              | Milk           | Common Allergens   |
| COMMON_EGGS              | Eggs           | Common Allergens   |
| COMMON_PEANUTS           | Peanuts        | Common Allergens   |
| COMMON_TREE_NUTS         | Tree nuts      | Common Allergens   |
| COMMON_SOY               | Soy            | Common Allergens   |
| COMMON_WHEAT             | Wheat          | Common Allergens   |
| COMMON_FISH              | Fish           | Common Allergens   |
| COMMON_SHELLFISH         | Shellfish      | Common Allergens   |
| COMMON_SESAME            | Sesame         | Common Allergens   |
| DAIRY_CASEIN             | Casein         | Dairy              |
| DAIRY_WHEY               | Whey           | Dairy              |
| DAIRY_LACTOSE            | Lactose        | Dairy              |
| DAIRY_BUTTER             | Butter         | Dairy              |
| DAIRY_CHEESE             | Cheese         | Dairy              |
| DAIRY_CREAM              | Cream          | Dairy              |
| DAIRY_YOGURT             | Yogurt         | Dairy              |
| DAIRY_ICE_CREAM          | Ice cream      | Dairy              |
| DAIRY_GHEE               | Ghee           | Dairy              |
| DAIRY_MILK_POWDER        | Milk powder    | Dairy              |
| DAIRY_COTTAGE_CHEESE     | Cottage cheese | Dairy              |
| DAIRY_SOUR_CREAM         | Sour cream     | Dairy              |
| DAIRY_BUTTERMILK         | Buttermilk     | Dairy              |
| DAIRY_KEFIR              | Kefir          | Dairy              |
| DAIRY_LACTALBUMIN        | Lactalbumin    | Dairy              |
| EGGS_CHICKEN             | Chicken eggs   | Eggs               |
| EGGS_DUCK                | Duck eggs      | Eggs               |
| EGGS_QUAIL               | Quail eggs     | Eggs               |
| EGGS_GOOSE               | Goose eggs     | Eggs               |
| EGGS_TURKEY              | Turkey eggs    | Eggs               |
| EGGS_WHITES              | Egg whites     | Eggs               |
| EGGS_YOLKS               | Egg yolks      | Eggs               |
| EGGS_POWDER              | Egg powder     | Eggs               |
| EGGS_LECITHIN            | Egg lecithin   | Eggs               |
| EGGS_PROTEINS            | Egg proteins   | Eggs               |
| EGGS_ALBUMIN             | Albumin        | Eggs               |
| EGGS_GLOBULIN            | Globulin       | Eggs               |
| EGGS_LIVETIN             | Livetin        | Eggs               |
| EGGS_OVOMUCOID           | Ovomucoid      | Eggs               |
| EGGS_OVALBUMIN           | Ovalbumin      | Eggs               |
| TREE_NUTS_ALMONDS        | Almonds        | Tree Nuts          |
| TREE_NUTS_BRAZIL         | Brazil nuts    | Tree Nuts          |
| TREE_NUTS_CASHEWS        | Cashews        | Tree Nuts          |
| TREE_NUTS_HAZELNUTS      | Hazelnuts      | Tree Nuts          |
| TREE_NUTS_MACADAMIA      | Macadamia nuts | Tree Nuts          |
| TREE_NUTS_PECANS         | Pecans         | Tree Nuts          |
| TREE_NUTS_PINE           | Pine nuts      | Tree Nuts          |
| TREE_NUTS_PISTACHIOS     | Pistachios     | Tree Nuts          |
| TREE_NUTS_WALNUTS        | Walnuts        | Tree Nuts          |
| TREE_NUTS_SHEA           | Shea nuts      | Tree Nuts          |
| TREE_NUTS_COCONUT        | Coconut        | Tree Nuts          |
| TREE_NUTS_CHESTNUTS      | Chestnuts      | Tree Nuts          |
| TREE_NUTS_HICKORY        | Hickory nuts   | Tree Nuts          |
| TREE_NUTS_BEECH          | Beechnuts      | Tree Nuts          |
| TREE_NUTS_GINKGO         | Ginkgo nuts    | Tree Nuts          |
| TREE_NUTS_LYCHEE         | Lychee nuts    | Tree Nuts          |
| TREE_NUTS_PILI           | Pili nuts      | Tree Nuts          |
| TREE_NUTS_CANDLE         | Candle nuts    | Tree Nuts          |
| TREE_NUTS_KOLA           | Kola nuts      | Tree Nuts          |
| TREE_NUTS_PARADISE       | Paradise nuts  | Tree Nuts          |
| PEANUTS_WHOLE            | Peanuts        | Peanuts            |
| PEANUTS_OIL              | Peanut oil     | Peanuts            |
| PEANUTS_FLOUR            | Peanut flour   | Peanuts            |
| PEANUTS_BUTTER           | Peanut butter  | Peanuts            |
| PEANUTS_PROTEIN          | Peanut protein | Peanuts            |
| PEANUTS_ARACHIS_OIL      | Arachis oil    | Peanuts            |
| PEANUTS_BEER_NUTS        | Beer nuts      | Peanuts            |
| PEANUTS_MIXED            | Mixed nuts     | Peanuts            |
| PEANUTS_NUT_MEAT         | Nut meat       | Peanuts            |
| PEANUTS_GROUND           | Ground nuts    | Peanuts            |
| PEANUTS_MONKEY           | Monkey nuts    | Peanuts            |
| PEANUTS_MANDELONAS       | Mandelonas     | Peanuts            |
| PEANUTS_NU_NUTS          | Nu-Nuts        | Peanuts            |
| PEANUTS_SPROUTS          | Peanut sprouts | Peanuts            |
| SOY_SOYBEANS             | Soybeans       | Soy                |
| SOY_SAUCE                | Soy sauce      | Soy                |
| SOY_TOFU                 | Tofu           | Soy                |
| SOY_TEMPEH               | Tempeh         | Soy                |
| SOY_MISO                 | Miso           | Soy                |
| SOY_NATTO                | Natto          | Soy                |
| SOY_EDAMAME              | Edamame        | Soy                |
| SOY_LECITHIN             | Soy lecithin   | Soy                |
| SOY_PROTEIN              | Soy protein    | Soy                |
| SOY_FLOUR                | Soy flour      | Soy                |
| SOY_MILK                 | Soy milk       | Soy                |
| SOY_NUTS                 | Soy nuts       | Soy                |
| SOY_SPROUTS              | Soy sprouts    | Soy                |
| SOY_TVP                  | Textured vegetable protein | Soy |
| SOY_HVP                  | Hydrolyzed vegetable protein | Soy |
| WHEAT_FLOUR              | Wheat flour    | Wheat              |
| WHEAT_BRAN               | Wheat bran     | Wheat              |
| WHEAT_GERM               | Wheat germ     | Wheat              |
| WHEAT_STARCH             | Wheat starch   | Wheat              |
| WHEAT_PROTEIN            | Wheat protein  | Wheat              |
| WHEAT_SEMOLINA           | Semolina       | Wheat              |
| WHEAT_DURUM              | Durum          | Wheat              |
| WHEAT_KAMUT              | Kamut          | Wheat              |
| WHEAT_SPELT              | Spelt          | Wheat              |
| WHEAT_TRITICALE          | Triticale      | Wheat              |
| WHEAT_VITAL_GLUTEN       | Vital wheat gluten | Wheat         |
| WHEAT_MODIFIED_STARCH    | Modified wheat starch | Wheat      |
| WHEAT_GRASS              | Wheat grass    | Wheat              |
| WHEAT_CRACKED            | Cracked wheat  | Wheat              |
| WHEAT_BULGUR             | Bulgur         | Wheat              |
| FISH_SALMON              | Salmon         | Fish               |
| FISH_TUNA                | Tuna           | Fish               |
| FISH_COD                 | Cod            | Fish               |
| FISH_HALIBUT             | Halibut        | Fish               |
| FISH_MACKEREL            | Mackerel       | Fish               |
| FISH_TROUT               | Trout          | Fish               |
| FISH_BASS                | Bass           | Fish               |
| FISH_FLOUNDER            | Flounder       | Fish               |
| FISH_SOLE                | Sole           | Fish               |
| FISH_HADDOCK             | Haddock        | Fish               |
| FISH_ANCHOVY             | Anchovy        | Fish               |
| FISH_SARDINES            | Sardines       | Fish               |
| FISH_TILAPIA             | Tilapia        | Fish               |
| FISH_MAHI_MAHI           | Mahi mahi      | Fish               |
| FISH_SWORDFISH           | Swordfish      | Fish               |
| FISH_PIKE                | Pike           | Fish               |
| FISH_PERCH       | Perch      | Fish     |
| FISH_SNAPPER     | Snapper    | Fish     |
| FISH_GROUPER     | Grouper    | Fish     |
| FISH_CATFISH     | Catfish    | Fish     |
| FISH_GELATIN     | Fish gelatin | Fish    |
| FISH_OIL         | Fish oil   | Fish     |
| FISH_SAUCE       | Fish sauce | Fish     |
| FISH_STOCK       | Fish stock | Fish     |
| FISH_CAVIAR      | Caviar     | Fish     |
| FISH_ROE         | Roe        | Fish     |
| SHELLFISH_SHRIMP | Shrimp     | Shellfish |
| SHELLFISH_CRAB   | Crab       | Shellfish |
| SHELLFISH_LOBSTER | Lobster   | Shellfish |
| SHELLFISH_CRAYFISH | Crayfish | Shellfish |
| SHELLFISH_PRAWNS | Prawns     | Shellfish |
| SHELLFISH_CLAMS  | Clams      | Shellfish |
| SHELLFISH_MUSSELS | Mussels   | Shellfish |
| SHELLFISH_OYSTERS | Oysters   | Shellfish |
| SHELLFISH_SCALLOPS | Scallops | Shellfish |
| SHELLFISH_SQUID  | Squid      | Shellfish |
| SHELLFISH_OCTOPUS | Octopus   | Shellfish |
| SHELLFISH_ABALONE | Abalone   | Shellfish |
| SHELLFISH_COCKLES | Cockles   | Shellfish |
| SHELLFISH_PERIWINKLES | Periwinkles | Shellfish |
| SHELLFISH_SEA_URCHIN | Sea urchin | Shellfish |
| SHELLFISH_BARNACLES | Barnacles | Shellfish |
| SHELLFISH_CRAWFISH | Crawfish | Shellfish |
| SHELLFISH_LANGOUSTINES | Langoustines | Shellfish |
| SHELLFISH_WHELKS | Whelks    | Shellfish |
| SESAME_SEEDS              | Sesame seeds          | Sesame            |
| SESAME_OIL               | Sesame oil            | Sesame            |
| SESAME_TAHINI            | Tahini                | Sesame            |
| SESAME_SESAMOL           | Sesamol               | Sesame            |
| SESAME_FLOUR             | Sesame flour          | Sesame            |
| SESAME_PASTE             | Sesame paste          | Sesame            |
| SESAME_GINGELLY_OIL      | Gingelly oil          | Sesame            |
| SESAME_BENNE             | Benne seeds           | Sesame            |
| SESAME_SALT              | Sesame salt           | Sesame            |
| SESAME_AQUA_LIBRA        | Aqua libra            | Sesame            |
| SESAME_SPROUTS           | Sesame sprouts        | Sesame            |
| FRUITS_APPLES            | Apples                | Fruits            |
| FRUITS_PEARS             | Pears                 | Fruits            |
| FRUITS_CHERRIES          | Cherries              | Fruits            |
| FRUITS_PEACHES           | Peaches               | Fruits            |
| FRUITS_PLUMS             | Plums                 | Fruits            |
| FRUITS_APRICOTS          | Apricots              | Fruits            |
| FRUITS_NECTARINES        | Nectarines            | Fruits            |
| FRUITS_STRAWBERRIES      | Strawberries          | Fruits            |
| FRUITS_RASPBERRIES       | Raspberries           | Fruits            |
| FRUITS_BLACKBERRIES      | Blackberries          | Fruits            |
| FRUITS_BLUEBERRIES       | Blueberries           | Fruits            |
| FRUITS_GRAPES            | Grapes                | Fruits            |
| FRUITS_BANANAS           | Bananas               | Fruits            |
| FRUITS_KIWI              | Kiwi                  | Fruits            |
| FRUITS_MANGO             | Mango                 | Fruits            |
| FRUITS_PINEAPPLE         | Pineapple             | Fruits            |
| FRUITS_PAPAYA            | Papaya                | Fruits            |
| FRUITS_FIGS              | Figs                  | Fruits            |
| FRUITS_DATES             | Dates                 | Fruits            |
| FRUITS_POMEGRANATE       | Pomegranate           | Fruits            |
| FRUITS_LYCHEE            | Lychee                | Fruits            |
| FRUITS_DRAGON_FRUIT      | Dragon fruit          | Fruits            |
| FRUITS_PASSION_FRUIT     | Passion fruit         | Fruits            |
| FRUITS_GUAVA             | Guava                 | Fruits            |
| FRUITS_PERSIMMON         | Persimmon             | Fruits            |
| FRUITS_AVOCADO           | Avocado               | Fruits            |
| FRUITS_ORANGES           | Oranges               | Fruits            |
| FRUITS_LEMONS            | Lemons                | Fruits            |
| FRUITS_LIMES             | Limes                 | Fruits            |
| FRUITS_GRAPEFRUIT        | Grapefruit            | Fruits            |
| FRUITS_TANGERINES        | Tangerines            | Fruits            |
| FRUITS_CLEMENTINES       | Clementines           | Fruits            |
| FRUITS_KUMQUATS          | Kumquats              | Fruits            |
| VEGETABLES_CELERY        | Celery                | Vegetables        |
| VEGETABLES_CARROTS       | Carrots               | Vegetables        |
| VEGETABLES_PARSNIPS      | Parsnips              | Vegetables        |
| VEGETABLES_TURNIPS       | Turnips               | Vegetables        |
| VEGETABLES_RUTABAGA      | Rutabaga              | Vegetables        |
| VEGETABLES_BEETS         | Beets                 | Vegetables        |
| VEGETABLES_RADISHES      | Radishes              | Vegetables        |
| VEGETABLES_POTATOES      | Potatoes              | Vegetables        |
| VEGETABLES_SWEET_POTATOES| Sweet potatoes        | Vegetables        |
| VEGETABLES_YAMS          | Yams                  | Vegetables        |
| VEGETABLES_ONIONS        | Onions                | Vegetables        |
| VEGETABLES_GARLIC        | Garlic                | Vegetables        |
| VEGETABLES_LEEKS         | Leeks                 | Vegetables        |
| VEGETABLES_SHALLOTS      | Shallots              | Vegetables        |
| VEGETABLES_CHIVES        | Chives                | Vegetables        |
| VEGETABLES_ASPARAGUS     | Asparagus             | Vegetables        |
| VEGETABLES_BROCCOLI      | Broccoli              | Vegetables        |
| VEGETABLES_CAULIFLOWER   | Cauliflower           | Vegetables        |
| VEGETABLES_BRUSSELS_SPROUTS | Brussels sprouts   | Vegetables        |
| VEGETABLES_CABBAGE       | Cabbage               | Vegetables        |
| VEGETABLES_KALE          | Kale                  | Vegetables        |
| VEGETABLES_COLLARD_GREENS| Collard greens        | Vegetables        |
| VEGETABLES_SPINACH       | Spinach               | Vegetables        |
| VEGETABLES_SWISS_CHARD   | Swiss chard           | Vegetables        |
| VEGETABLES_LETTUCE       | Lettuce               | Vegetables        |
| VEGETABLES_ARUGULA       | Arugula               | Vegetables        |
| VEGETABLES_WATERCRESS    | Watercress            | Vegetables        |
| VEGETABLES_MUSHROOMS     | Mushrooms             | Vegetables        |
| VEGETABLES_TOMATOES      | Tomatoes              | Vegetables        |
| VEGETABLES_PEPPERS       | Peppers               | Vegetables        |
| VEGETABLES_EGGPLANT      | Eggplant              | Vegetables        |
| VEGETABLES_ZUCCHINI      | Zucchini              | Vegetables        |
| VEGETABLE_CUCUMBER       | Cucumber              | Vegetables        |
| VEGETABLE_PUMPKIN        | Pumpkin               | Vegetables        |
| VEGETABLE_SQUASH         | Squash                | Vegetables        |
| VEGETABLE_ARTICHOKES     | Artichokes            | Vegetables        |
| VEGETABLE_BAMBOO_SHOOTS  | Bamboo shoots         | Vegetables        |
| VEGETABLE_BEAN_SPROUTS   | Bean sprouts          | Vegetables        |
| VEGETABLE_CORN           | Corn                  | Vegetables        |
| VEGETABLE_GREEN_BEANS    | Green beans           | Vegetables        |
| VEGETABLE_PEAS           | Peas                  | Vegetables        |
| VEGETABLE_JICAMA         | Jicama                | Vegetables        |
| VEGETABLE_KOHLRABI       | Kohlrabi              | Vegetables        |
| VEGETABLE_OKRA           | Okra                  | Vegetables        |
| LEGUME_BLACK_BEANS         | Black beans          | Legumes           |
| LEGUME_KIDNEY_BEANS        | Kidney beans         | Legumes           |
| LEGUME_NAVY_BEANS          | Navy beans           | Legumes           |
| LEGUME_PINTO_BEANS         | Pinto beans          | Legumes           |
| LEGUME_WHITE_BEANS         | White beans          | Legumes           |
| LEGUME_LIMA_BEANS          | Lima beans           | Legumes           |
| LEGUME_FAVA_BEANS          | Fava beans           | Legumes           |
| LEGUME_CHICKPEAS           | Chickpeas            | Legumes           |
| LEGUME_LENTILS             | Lentils              | Legumes           |
| LEGUME_SPLIT_PEAS          | Split peas           | Legumes           |
| LEGUME_MUNG_BEANS          | Mung beans           | Legumes           |
| LEGUME_ADZUKI_BEANS        | Adzuki beans         | Legumes           |
| LEGUME_BLACK_EYED_PEAS     | Black-eyed peas      | Legumes           |
| LEGUME_CANNELLINI_BEANS    | Cannellini beans     | Legumes           |
| LEGUME_GREAT_NORTHERN_BEANS| Great Northern beans | Legumes           |
| LEGUME_LUPINI_BEANS        | Lupini beans         | Legumes           |
| LEGUME_SOYBEANS            | Soybeans             | Legumes           |
| LEGUME_GREEN_PEAS          | Green peas           | Legumes           |
| LEGUME_SNOW_PEAS           | Snow peas            | Legumes           |
| LEGUME_SUGAR_SNAP_PEAS     | Sugar snap peas      | Legumes           |
| GRAIN_BARLEY               | Barley               | Grains            |
| GRAIN_BUCKWHEAT            | Buckwheat            | Grains            |
| GRAIN_CORN                 | Corn                 | Grains            |
| GRAIN_MILLET               | Millet               | Grains            |
| GRAIN_OATS                 | Oats                 | Grains            |
| GRAIN_QUINOA               | Quinoa               | Grains            |
| GRAIN_RICE                 | Rice                 | Grains            |
| GRAIN_RYE                  | Rye                  | Grains            |
| GRAIN_SORGHUM              | Sorghum              | Grains            |
| GRAIN_TEFF                 | Teff                 | Grains            |
| GRAIN_WILD_RICE            | Wild rice            | Grains            |
| GRAIN_AMARANTH             | Amaranth             | Grains            |
| GRAIN_FARRO                | Farro                | Grains            |
| GRAIN_FREEKEH              | Freekeh              | Grains            |
| GRAIN_JOBS_TEARS           | Job's tears          | Grains            |
| GRAIN_KANIWA               | Kaniwa               | Grains            |
| GRAIN_EINKORN              | Einkorn              | Grains            |
| GRAIN_EMMER                | Emmer                | Grains            |
| SEED_SUNFLOWER             | Sunflower seeds      | Seeds             |
| SEED_PUMPKIN               | Pumpkin seeds        | Seeds             |
| SEED_FLAX                  | Flax seeds           | Seeds             |
| SEED_CHIA                  | Chia seeds           | Seeds             |
| SEED_HEMP                  | Hemp seeds           | Seeds             |
| SEED_POPPY                 | Poppy seeds          | Seeds             |
| SEED_CARAWAY               | Caraway seeds        | Seeds             |
| SEED_FENNEL                | Fennel seeds         | Seeds             |
| SEED_MUSTARD               | Mustard seeds        | Seeds             |
| SEED_CUMIN                 | Cumin seeds          | Seeds             |
| SEED_CORIANDER             | Coriander seeds      | Seeds             |
| SEED_CELERY                | Celery seeds         | Seeds             |
| SEED_ANISE                 | Anise seeds          | Seeds             |
| SEED_NIGELLA               | Nigella seeds        | Seeds             |
| SEED_FENUGREEK             | Fenugreek seeds      | Seeds             |
| SPICE_BLACK_PEPPER         | Black pepper         | Spices and Herbs  |
| SPICE_WHITE_PEPPER         | White pepper         | Spices and Herbs  |
| SPICE_RED_PEPPER           | Red pepper           | Spices and Herbs  |
| SPICE_CHILI_POWDER         | Chili powder         | Spices and Herbs  |
| SPICE_PAPRIKA              | Paprika              | Spices and Herbs  |
| SPICE_CINNAMON             | Cinnamon             | Spices and Herbs  |
| SPICE_CLOVES               | Cloves               | Spices and Herbs  |
| SPICE_NUTMEG               | Nutmeg               | Spices and Herbs  |
| SPICE_MACE                 | Mace                 | Spices and Herbs  |
| SPICE_GINGER               | Ginger               | Spices and Herbs  |
| SPICE_TURMERIC             | Turmeric             | Spices and Herbs  |
| SPICE_CARDAMOM             | Cardamom             | Spices and Herbs  |
| SPICE_CORIANDER            | Coriander            | Spices and Herbs  |
| SPICE_CUMIN                | Cumin                | Spices and Herbs  |
| SPICE_FENNEL               | Fennel               | Spices and Herbs  |
| SPICE_FENUGREEK            | Fenugreek            | Spices and Herbs  |
| SPICE_GARLIC_POWDER        | Garlic powder        | Spices and Herbs  |
| SPICE_ONION_POWDER         | Onion powder         | Spices and Herbs  |
| SPICE_MUSTARD_POWDER       | Mustard powder       | Spices and Herbs  |
| SPICE_SAFFRON              | Saffron              | Spices and Herbs  |
| HERB_BASIL                 | Basil                | Spices and Herbs  |
| HERB_OREGANO               | Oregano              | Spices and Herbs  |
| HERB_THYME                 | Thyme                | Spices and Herbs  |
| HERB_ROSEMARY              | Rosemary             | Spices and Herbs  |
| HERB_SAGE                  | Sage                 | Spices and Herbs  |
| HERB_MARJORAM              | Marjoram             | Spices and Herbs  |
| HERB_TARRAGON              | Tarragon             | Spices and Herbs  |
| HERB_DILL                  | Dill                 | Spices and Herbs  |
| HERB_MINT                  | Mint                 | Spices and Herbs  |
| HERB_PARSLEY               | Parsley              | Spices and Herbs  |
| HERB_CILANTRO              | Cilantro             | Spices and Herbs  |
| HERB_BAY_LEAVES            | Bay leaves           | Spices and Herbs  |
| SPICE_CARAWAY              | Caraway              | Spices and Herbs  |
| SPICE_JUNIPER_BERRIES      | Juniper berries      | Spices and Herbs  |
| SPICE_WASABI               | Wasabi               | Spices and Herbs  |
| SPICE_HORSERADISH          | Horseradish          | Spices and Herbs  |
| ADDITIVE_MSG               | MSG                  | Additives         |
| ADDITIVE_SULFITES          | Sulfites             | Additives         |
| ADDITIVE_NITRATES          | Nitrates             | Additives         |
| ADDITIVE_NITRITES          | Nitrites             | Additives         |
| ADDITIVE_BHA               | BHA                  | Additives         |
| ADDITIVE_BHT               | BHT                  | Additives         |
| ADDITIVE_CARRAGEENAN       | Carrageenan          | Additives         |
| ADDITIVE_ANNATTO           | Annatto              | Additives         |
| ADDITIVE_TARTRAZINE        | Tartrazine           | Additives         |
| ADDITIVE_RED_DYE_40        | Red dye 40           | Additives         |
| ADDITIVE_YELLOW_DYE_5      | Yellow dye 5         | Additives         |
| ADDITIVE_YELLOW_DYE_6      | Yellow dye 6         | Additives         |
| ADDITIVE_BLUE_DYE_1        | Blue dye 1           | Additives         |
| ADDITIVE_ASPARTAME         | Aspartame            | Additives         |
| ADDITIVE_SUCRALOSE         | Sucralose            | Additives         |
| ADDITIVE_SACCHARIN         | Saccharin            | Additives         |
| ADDITIVE_STEVIA            | Stevia               | Additives         |
| ADDITIVE_ACESULFAME_K      | Acesulfame potassium | Additives         |
| ADDITIVE_GUAR_GUM          | Guar gum             | Additives         |
| ADDITIVE_XANTHAN_GUM       | Xanthan gum          | Additives         |
| ADDITIVE_CARBOXYMETHYL_CELLULOSE | Carboxymethyl cellulose | Additives |
| ADDITIVE_MALTODEXTRIN      | Maltodextrin         | Additives         |
| ADDITIVE_MODIFIED_FOOD_STARCH | Modified food starch | Additives      |
| ADDITIVES_SODIUM_BENZOATE     | Sodium benzoate           | Additives         |
| ADDITIVES_POTASSIUM_SORBATE   | Potassium sorbate         | Additives         |
| ADDITIVES_PROPYLENE_GLYCOL    | Propylene glycol          | Additives         |
| ADDITIVES_POLYSORBATES        | Polysorbates              | Additives         |
| ADDITIVES_SODIUM_NITRATE      | Sodium nitrate            | Additives         |
| ADDITIVES_CALCIUM_PROPIONATE  | Calcium propionate        | Additives         |
| ADDITIVES_PHOSPHORIC_ACID     | Phosphoric acid           | Additives         |
| OTHERS_YEAST                  | Yeast                     | Others            |
| OTHERS_GELATIN                | Gelatin                   | Others            |
| OTHERS_CARMINE                | Carmine                   | Others            |
| OTHERS_COCHINEAL_EXTRACT      | Cochineal extract         | Others            |
| OTHERS_ROYAL_JELLY            | Royal jelly               | Others            |
| OTHERS_PROPOLIS               | Propolis                  | Others            |
| OTHERS_BEE_POLLEN             | Bee pollen                | Others            |
| OTHERS_HONEY                  | Honey                     | Others            |
| OTHERS_LATEX_FRUITS           | Latex fruits              | Others            |
| OTHERS_ALPHA_GAL              | Alpha-gal (meat allergy)  | Others            |
| OTHERS_FOOD_ENZYMES           | Food enzymes              | Others            |
| OTHERS_LUPIN                  | Lupin                     | Others            |
| OTHERS_MOLLUSKS               | Mollusks                  | Others            |
| OTHERS_BUCKWHEAT              | Buckwheat                 | Others            |
| OTHERS_MARSHMALLOW_ROOT       | Marshmallow root          | Others            |
| OTHERS_CHICORY_ROOT           | Chicory root              | Others            |
| OTHERS_TRAGACANTH             | Tragacanth                | Others            |
| OTHERS_ACACIA_GUM             | Acacia gum                | Others            |
| OTHERS_CAROB                  | Carob                     | Others            |
| OTHERS_MESQUITE               | Mesquite                  | Others            |



# ----- B2B

# Account

## Create a new provider into our database

```graphql
mutation InsertProvider(
  $uid: String!,
  $email: String!, 
  $first_name: String!, 
  $middle_name: String="",
  $last_name: String!,
  $npi: String!
  $provider_type: String!, 
  $designation: String = "", 
  $org_name: String = ""
) {
  insert_provider(objects: {
    uid: $uid,
    email: $email, 
    first_name: $first_name, 
    middle_name: $middle_name, 
    last_name: $last_name, 
    npi: $npi, 
    provider_type: $provider_type, 
    designation: $designation,
    provider_org: {
      data: {
        org_name: $org_name
      }
    }
  }) {
    returning {
      id
    }
  }
}
```

> Example input parameter:

```graphql
{
  "uid":"testuid",
  "email": "kyungha329@gmail.com",
  "first_name": "Kay",
  "middle_name": "",
  "last_name": "Lim",
  "npi": "0000012345",
  "provider_type": "founder",
  "designation": "Dr.",
  "org_name":"Kay Lim LLC"
}
```

### Parameters:

Parameter         | Type        | Description
----------------- | ----------- | -----------
uid               | text        | User ID provided by Cognito
email             | text        | Work email
first_name        | text        | First name
middle_name       | text        | Middle name. Enter empty string if no middle name
last_name         | text        | Last name
npi               | text        | 10 digit NPI number, verified through NIH API
provider_type     | text        | Provider Type registered and verified through NIH API
designation       | text        | Open field. Dr, RDN, etc... for providers to enter
org_name          | text        | Organization name. Tell the user to enter their own name if they don't have a registered business entity

Make sure to store the output ID so that we can save the plan that the provider chooses in the next step.

<aside class="warning">
The email ID should be unique. If it returns unique violation error, show a warning that this account already exists with the email. Use another email address or log in with the existing account.
</aside>

```graphql
Example error format. Please check the error format directly in the code.

{
  "errors": [
    {
      "message": "Uniqueness violation. duplicate key value violates unique constraint \"provider_email_key\"",
      "extensions": {
        "path": "$.selectionSet.insert_provider.args.objects[0]",
        "code": "constraint-violation"
      }
    }
  ]
}
```


## Select a plan

```graphql
mutation SelectProviderPlan(
  $provider_id: uuid!, 
  $patient_limit: Int!,
  $seat_limit: Int!) {
  update_provider_org(where: {
    providers: {id: {_eq: $provider_id}}
  }, _set: {
    patient_limit: $patient_limit, 
    seat_limit: $seat_limit, 
    plan_status: ACTIVE
  }) {
    affected_rows
  }
}
```

> Example input parameter:

```graphql
{
  "provider_id": "da574170-684f-4da3-b399-524118b43e64",
  "patient_limit": 10,
  "seat_limit":2
}
```

### Parameters:

Parameter         | Type        | Description
----------------- | ----------- | -----------
provider_id       | text        | Provider ID generated by the database (e.g. id from provider table)
seat_limit        | Integer     | Number of seats in the selected plan that the provider selected
patient_limit     | Integer     | Number of patients in the selected plan that the provider selected



## Cancel a plan

```graphql
mutation CancelPlan(
  $provider_id: uuid!,
  $exp_date: timestamptz
) {
  update_provider_org(where: {
    providers: {id: {_eq: $provider_id}}
  }, _set: {
    expired_at: $exp_date,
    plan_status: DEACTIVATED
  }) {
    affected_rows
  }
}
```

> Example input parameter:

```graphql
{
  "provider_id": "da574170-684f-4da3-b399-524118b43e64",
  "exp_date": "2024-12-03"
}
```

### Parameters:

Parameter         | Type        | Description
----------------- | ----------- | -----------
provider_id       | text        | Provider ID generated by the database (e.g. id from provider table)
exp_date          | Timestamp   | Current plan expiration date


Make a note of the plan status. We are marking DEACTIVATED but the service should be available until the exp_date.

# Patient Intake

## Invite Patient

```graphql
mutation InvitePatient(
  $provider_id: uuid!, 
  $email: String!, 
  $onboarding_message:String!, 
  $name: String!
) {
  insert_provider_patient(objects: {
    patient_email: $email, 
    provider_id: $provider_id,
    onboarding_message: $onboarding_message,
    onboarding_name: $name,
  }){
    returning {
      id
    }
  }
}
```

> Example input parameter:

```graphql
{
  "name": "Kay Lim"
  "email": "kay@heartfulsprout.com",
  "provider_id": "9301c143-0c83-44ff-b248-4495f529306f",
  "onboarding_message": "Hello!",
}
```

Make sure to save the provider_patient ID (returning data). Use this for invite link?

### Parameters:

Parameter         | Type        | Description
----------------- | ----------- | -----------
provider_id       | text        | Provider ID generated by the database (e.g. id from provider table)
name              | text        | Name of the recipient
email             | text        | Email of the recipient
onboarding_message | text       | Onboarding message to send


## Guardian and PC Registration

```graphql
mutation RegisterGuardianAndPC(
  $registrationName: String, 
  $registrationRel: String, 
  $guardians: [patient_guardian_insert_input!]!, 
  $pc_address: String = "", 
  $pc_first_name: String = "", 
  $pc_last_name: String = "", 
  $pc_occupation: String = "", 
  $pc_title: String = ""
) {
  insert_patient(objects: {
    reg_name: $registrationName, 
    reg_rel: $registrationRel, 
    patient_pc: {data: {
      pc_first_name: $pc_first_name, 
      pc_last_name: $pc_last_name, 
      pc_title: $pc_title, 
      pc_address: $pc_address, 
      pc_occupation: $pc_occupation}
    }
  }) {
    returning {
      id
    }
  }
  insert_patient_guardian(objects: $guardians) {
    returning {
      id
      guardian_type
    }
  }
}

```

> Example Input Parameter:

```graphql
{
  "registrationName": "Kay Lim",
  "registrationRel": "Mother",
  "pc_address": "somewhere address",
  "pc_first_name": "Beth",
  "pc_last_name": "Conlon",
  "pc_occupation": "RDN",
  "pc_title": "RDN",
  "guardians": [
    {
      "first_name": "Kay",
      "last_name": "Lim",
      "address":"1035 test address, chicago, IL 60606",
      "birthday":"1986/03/29",
      "home_number": null,
      "mobile_number": null,
      "gender":"FEMALE",
      "relationship": "Mother",
      "relationship_status": "Married",
      "occupation": "Founder",
      "work_hr_per_wk": "60",
      "note":"very healthy",
      "guardian_type": "PRIMARY"
    },
    {
      "first_name": "TESTTEST",
      "last_name": "TEST",
      "address":"1035 test address, chicago, IL 60606",
      "birthday":"1987/03/12",
      "home_number": null,
      "mobile_number": null,
      "gender":"MALE",
      "relationship": "Father",
      "relationship_status": "Married",
      "occupation": "Founder",
      "work_hr_per_wk": "60",
      "note": null,
      "guardian_type": "EMERGENCY"
    }
  ]
}
```



### Parameters:

Parameter            | Type        | Description
-------------------- | ----------- | -----------
registrationName     | text        | The name of the person filling out the form
registrationRel      | text        | Relationship of this person to the patient.
guardians             | list of objects | See the example parameters to see the field names
pc_address            | text        | Address of primary care
pc_first_name         | text        | First name of primary care doctor
pc_last_name          | text        | Last name of primary care doctor
pc_occupation         | text        | Occupation of the primary care doctor
pc_title              | text        | MD, RDN, etc. title of the doctor

<aside>
This step must happen before registrating patient. Make sure to save the output because it will be needed for patient registration.
</aside>

```graphql
{
  "data": {
    "insert_patient": {
      "returning": [
        {
          "id": "814eed3d-c6a3-4efe-a392-74ba3f0b59b4"
        }
      ]
    },
    "insert_patient_guardian": {
      "returning": [
        {
          "id": "4c00c6cb-571b-4f75-92c2-3114b31a8752",
          "guardian_type": "PRIMARY"
        },
        {
          "id": "180a389c-04cb-4b54-b59f-3a47d6b7515b",
          "guardian_type": "EMERGENCY"
        }
      ]
    }
  }
}
```
The output should look like the example on the right. We need all the IDs in the next query.


## Patient Registration

```graphql
mutation RegisterPatient(
  $id: uuid!, 
  $provider_patient_id: uuid!,
  $first_name: String!, 
  $middle_name: String = "", 
  $last_name: String!, 
  $preferred_name: String = "", 
  $birthday: String!, 
  $sex: String!, 
  $birth_country: String!, 
  $home_address: String!, 
  $billing_address: String!, 
  $guardian1: uuid!, 
  $guardian2: uuid = null, 
  $emergency_contact: uuid!, 
  $referred_by: String = "", 
  $referral_reason: String = "", 
  $source: String = "", 
  $contact_permission: Boolean = false, 
  $primary_care: uuid = null) {
    update_provider_patient(where:{id: {_eq:$provider_patient_id}}, _set: {patient_id:$id}){
      returning {
        id
      }
    }
  update_patient(
    where: {id: {_eq: $id}}, 
    _set: {
      first_name: $first_name, 
      middle_name: $middle_name, 
      last_name: $last_name, 
      preferred_name: $preferred_name, 
      birthday: $birthday, 
      sex: $sex, 
      birth_country: $birth_country, 
      home_address: $home_address, 
      billing_address: $billing_address, 
      guardian1: $guardian1, 
      guardian2: $guardian2, 
      emergency_contact: $emergency_contact, 
      referred_by: $referred_by, 
      referral_reason: $referral_reason, 
      source: $source, 
      primary_care: $primary_care, 
      contact_permission: $contact_permission
    })
    {
      returning {
        id
      }
    }
}

```

> Example input parameter:

```graphql
{
  "id": "814eed3d-c6a3-4efe-a392-74ba3f0b59b4",
  "first_name": "Hannah",
	"middle_name": null,
  "last_name": "Kim",
  "preferred_name": null,
  "birthday": "2020-04-14",
  "sex": "FEMALE",
  "birth_country": "USA",
  "home_address": "1035 somewhere in the US",
  "billing_address": "1035 somewhere in the US",
  "guardian1": "4c00c6cb-571b-4f75-92c2-3114b31a8752",
  "guardian2": "4c00c6cb-571b-4f75-92c2-3114b31a8752",
  "emergency_contact": "180a389c-04cb-4b54-b59f-3a47d6b7515b",
  "referred_by": null,
  "referral_reason": "Picky eating",
  "source": "Social Media",
  "contact_permission": true,
  "primary_care": null,
  "provider_patient_id": "4994955a-9745-4847-a323-848028bff92a"
}

```

### Parameters:

Parameter         | Type        | Description
----------------- | ----------- | -----------
id                | uuid        | Patient ID generated from Guardian Registration (step above)
first_name        | text        | First name of the child
middle_name       | text        | Middle name of the child
last_name         | text        | Last name of the child
preferred_name    | text        | Preferred name of the child
birthday          | text        | Birthday. Note that this is a text field. Please use a datepicker to ensure data format consistancy
sex               | text        | Sex (when born). MALE or FEMALE
birth_country     | text        | Country of birth
home_address      | text        | Concatenated string of home address
billing_address   | text        | Concatenated string of billing address
guardian1         | uuid        | ID of primary guardian
guardian2         | uuid        | ID of second guardian
emergency_contact | uuid        | ID emergency contact
referred_by       | text        | Referred by
referral_reason   | text        | Reason for referral
source            | text        | Where did they hear about the provider?
primary_care      | uuid        | ID of primary care provider
contact_permission| boolean     | Whether the user agrees to get contacted


## Grant Premium Access to Patient

<code>
https://api.revenuecat.com/v1/subscribers/<patient_email here>/entitlements/provider_referred/promotional
</code>

- API method: POST
- Headers must include <code>Authorization: Bearer REVENUE_CAT_API_KEY</code>
- Body must include:

{

  "start_time_ms": 1709195668093,   // current time in milliseconds
  
  "end_time_ms": 1738389600000      // let's just add December 31, 2026 for now.

}


## Save Registration File
```graphql
mutation SaveRegistrationPDF(
  $provider_patient_id: uuid!,
  $reg_file:String!) {
  update_provider_patient(
    where: {id: {_eq: $provider_patient_id}}, 
    _set: {reg_file: $reg_file, reg_status: COMPLETE}){
      affected_rows
    }
}
```


> Example input parameter:

```graphql
{
  "provider_patient_id": "077a3f74-cbc5-49ae-909c-3fc15f511c90",
  "reg_file": "S3BUCKETADDRESS"
}
```

### Parameters:

Parameter         | Type        | Description
----------------- | ----------- | -----------
provider_patient_id | uuid      | ID of provider_patient table
reg_file          | text        | File key in S3 bucket


<aside>
Let's come back to the uploading files later.
</aside>


## Insurance Form
```graphql
mutation RegisterInsurance(
  $first_name: String!,
  $last_name: String!, 
  $policy_holder: String!, 
  $birthday: String!, 
  $gender: String!, 
  $phone_number: String!, 
  $address: String!, 
  $insurance_company: String!, 
  $payer_id: String!, 
  $coverage_type: String!, 
  $member_id: String!, 
  $plan_id: String!, 
  $group_id: String!, 
  $copay: String!, 
  $deductible: String!, 
  $primary_care: uuid = "", 
  $authorization_agreed: Boolean!
) {
  insert_patient_insurance(objects: {
    first_name: $first_name, 
    last_name: $last_name, 
    policy_holder: $policy_holder, 
    birthday: $birthday, 
    gender: $gender, 
    phone_number: $phone_number, 
    address: $address, 
    insurance_company: $insurance_company, 
    payer_id: $payer_id, 
    coverage_type: $coverage_type, 
    member_id: $member_id, 
    plan_id: $plan_id, 
    group_id: $group_id, 
    copay: $copay, 
    deductible: $deductible, 
    primary_care: $primary_care, 
    authorization_agreed: $authorization_agreed
  }){
    returning{
      id
    }
  }
}
```

> Example input

```graphql
{
  "first_name": "Kay",
  "last_name": "Lim",
  "policy_holder": "Kay Lim",
  "birthday": "1987-04-14",
  "gender": "FEMALE",
  "phone_number": "000-000-0000",
  "address": "somewhere, city, state, 00000",
  "insurance_company": "Insurance company",
  "payer_id": "00000",
  "coverage_type": "Premium",
  "member_id": "00000",
  "plan_id": "00000",
  "group_id": "00000",
  "copay": "50",
  "deductible":"1000",
  "primary_care": null,
  "authorization_agreed": true
}
```


## Save Insurance File
```graphql
mutation SaveInsurancePDF(
  $provider_patient_id: uuid!,
  $ins_id: uuid!,
  $ins_file: String!) {
  update_provider_patient(
    where: {id: {_eq: $provider_patient_id}}, 
    _set: {ins_id: $ins_id, ins_file: $ins_file, ins_status: COMPLETE}){
      affected_rows
    }
}
```


> Example input parameter:

```graphql
{
  "provider_patient_id": "077a3f74-cbc5-49ae-909c-3fc15f511c90",
  "ins_id": "077a3f74-cbc5-49ae-909c-3fc15f511c90",
  "ins_fild": "S3BUCKETADDRESS"
}
```

### Parameters:

Parameter         | Type        | Description
----------------- | ----------- | -----------
provider_patient_id | uuid      | ID of provider_patient table
ins_id            | uuid        | ID from patient_ins table
ins_file          | text        | File key in S3 bucket


<aside>
Let's come back to the uploading files later.
</aside>


## HIPAA Consent
```graphql
mutation HIPAAConsent(
  $first_name: String!,
  $last_name: String!, 
  $relationship:String!
) {
  insert_patient_hipaa(objects: {
    first_name: $first_name, 
    last_name: $last_name, 
    relationship: $relationship
  }){
    returning{
      id
    }
  }
}
```

> Example input

```graphql
{
  "first_name": "Kay",
  "last_name": "Lim",
  "relationship": "Mother"
}
```

## Save HIPAA File
```graphql
mutation SaveHIPAAPDF(
  $provider_patient_id: uuid!,
  $hipaa_id: uuid!,
  $hipaa_file: String!) {
  update_provider_patient(
    where: {id: {_eq: $provider_patient_id}}, 
    _set: {hipaa_id: $hipaa_id, hipaa_file: $hipaa_file, hipaa_status: COMPLETE}){
      affected_rows
    }
}
```


> Example input parameter:

```graphql
{
  "provider_patient_id": "077a3f74-cbc5-49ae-909c-3fc15f511c90",
  "hipaa_id": "077a3f74-cbc5-49ae-909c-3fc15f511c90",
  "hipaa_fild": "S3BUCKETADDRESS"
}
```

### Parameters:

Parameter         | Type        | Description
----------------- | ----------- | -----------
provider_patient_id | uuid      | ID of provider_patient table
hipaa_id          | uuid        | ID from patient_hipaa table
hipaa_file        | text        | File key in S3 bucket


<aside>
Let's come back to the uploading files later.
</aside>


# Session

## Start and save transcript

```graphql
mutation SaveTranscript(
  $start_time: timestamptz, 
  $end_time: timestamptz, 
  $transcript_type: String, 
  $transcript: String, 
  $provider_patient_id: uuid!) {
  insert_provider_session(objects: {
    start_time: $start_time, 
    end_time: $end_time, 
    transcript_type: $transcript_type, 
    transcript: $transcript, 
    provider_patient_id: $provider_patient_id
  }){
    returning {
      id
    }
  }
}
```

> Example input

```graphql
{
  "start_time": "2024-12-10T13:45:00.000Z",
  "end_time": "2024-12-10T14:30:00.000Z",
  "transcript_type": "live",
  "transcript": "transcript here",
  "provider_patient_id": "077a3f74-cbc5-49ae-909c-3fc15f511c90"
}
```

## Save note taking

```graphql
mutation SaveSessionNotes(
  $id: uuid!, 
  $intervention_notes: String = "", 
  $intervention_summary: String = "", 
  $notes: String = "", 
  $notes_type: String = "", 
  $patient_notes: String = ""
) {
  update_provider_session(
    where: {id: {_eq: $id}}, 
    _set: {
      intervention_notes: $intervention_notes, 
      intervention_summary: $intervention_summary, 
      notes: $notes, 
      notes_type: $notes_type, 
      patient_notes: $patient_notes
    }
  ){
    affected_rows
  }
}

```

> Example input

```graphql
{
  "id": "7738efac-a2da-4393-8ec4-57a9c6a8a878",
  "intervention_notes": "intervention notes",
  "intervention_summary": "interventino summary",
  "notes": "notes",
  "notes_type": "SOAP",
  "patient_notes": "patient notes"
}
```

# Nutrition Perscription

## Enter a perscription
```graphql
mutation NutritionPerscription(
  $objects: [prescription_nutrition_insert_input!]!) {
  insert_prescription_nutrition(objects: $objects) {
    returning {
      id
    }
  }
}
```

> Example input
```graphql
{
  "objects": [{
    "activity_level": "ACTIVE",
    "added_sugar_G": 0,
    "age":180,
    "calcium_MG": 1,
    "calories_KCAL": 800,
    "carbohydrate_G": 1,
    "cholesterol_MG": 0,
    "choline_MG": 1,
    "folatedfe_UG": 1,
    "height_cm": 80,
    "iron_MG": 1,
    "magnesium_MG": 1,
    "niacin_MG": 1,
    "omega3_G": 1,
    "omega6_G": 1,
    "phosphorus_MG": 1,
    "potassium_MG": 1,
    "protein_G": 1,
    "provider_patient_id": "077a3f74-cbc5-49ae-909c-3fc15f511c90",
    "provider_session_id":"7738efac-a2da-4393-8ec4-57a9c6a8a878",
    "riboflavin_MG": 1,
    "saturated_fatty_acids_G": 1,
    "sodium_MG": 1,
    "thiamin_MG": 1,
    "total_dietary_fiber_G": 1,
    "total_fatty_acids_G": 1,
    "total_lipid_G": 1,
    "total_sugar_G": 1,
    "vitamina_UG": 1,
    "vitaminb12_UG": 1,
    "vitaminc_MG": 1,
    "vitamind_UG": 1,
    "vitamine_MG": 1,
    "vitamink_UG": 1,
    "weight_kg": 8,
    "zinc_MG": 1
  }]
}
```

Note that this is typically within the session flow, but clinicians should be able to adjust nutrition outside the session.


# Meal Plan Assignment

## Enter a meal plan

```graphql
mutation MealPlanPerscription($objects: [prescription_meal_plan_insert_input!] = {}) {
  insert_prescription_meal_plan(objects: $objects){
    returning{
      id
    }
  }
}
```

> Example input 

```graphql
{
  "objects": [{
    "provider_patient_id": "077a3f74-cbc5-49ae-909c-3fc15f511c90",
    "provider_session_id":"7738efac-a2da-4393-8ec4-57a9c6a8a878",
    "recipe_id": "58fc5d0a-93e1-4762-b890-ffc17c9a4179",
    "planned_at": "2024-12-10"
  },{
    "provider_patient_id": "077a3f74-cbc5-49ae-909c-3fc15f511c90",
    "provider_session_id":"7738efac-a2da-4393-8ec4-57a9c6a8a878",
    "recipe_id": "58fc5d0a-93e1-4762-b890-ffc17c9a4179",
    "planned_at": "2024-12-11"
  },{
    "provider_patient_id": "077a3f74-cbc5-49ae-909c-3fc15f511c90",
    "provider_session_id":"7738efac-a2da-4393-8ec4-57a9c6a8a878",
    "recipe_id": "58fc5d0a-93e1-4762-b890-ffc17c9a4179",
    "planned_at": "2024-12-12"
  }]
}
```

# Dashboard

## Get patient list for overview
```graphql
query GetPatientList($provider_id: uuid = "") {
  patient(where: {provider_patients: {provider: {id: {_eq: $provider_id}}}}) {
    id
    first_name
    preferred_name
    middle_name
    last_name
    birthday
    sex
    primary_diagnosis
  }
}
```

## Patient Detail
```graphql
query GetPatientDetail($patient_id: uuid = "") {
  patient(where: {id: {_eq: $patient_id}}) {
    birth_country
    birthday
    home_address
    last_name
    middle_name
    patientGuardianByEmergencyContact {
      first_name
      last_name
      home_number
      mobile_number
      note
      relationship
    }
    patientGuardianByGuardian2 {
      first_name
      last_name
      note
      birthday
      email
      mobile_number
      guardian_type
      gender
      relationship
      relationship_status
      work_hr_per_wk
      occupation
    }
    patient_guardian {
      address
      email
      birthday
      first_name
      gender
      guardian_type
      last_name
      mobile_number
      note
      occupation
      relationship
      relationship_status
    }
    preferred_name
    preferred_pronouns
    primary_diagnosis
    referral_reason
    referred_by
    sex
    source
  }
}

```

> Example input

```graphql
{
  "patient_id": "814eed3d-c6a3-4efe-a392-74ba3f0b59b4"
}
```


# --- DONE