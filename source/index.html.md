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

Note that environment variables are not explicitly written. 

<aside class="notice">
API calls require either admin secret or user token uses API keys to allow access to the API. Please let Kay know if you need those variables and don’t have them.
</aside>


The userIDToken is automatically generated in jwt format when authenticated through AWS Cognito. The AWS Cognito returns accessToken and idToken, and the idToken is the token to be used for Hasura. It should contain:

* role: user
* X-Hasura-User-ID: <code>user ID from Cognito</code>


# Apollo Provider (React Native)

## Create an Apollo Client

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

```javascript
import {GQL_COMMENT} from '@env';

const [gqlFunction, {data, error}] = useQuery(GQL_COMMAND);

gqlFunction({
  variables: inputParams
})
```


## useMutation

```javascript
import {GQL_COMMENT} from '@env';

const [gqlFunction, {data, error}] = useMutation(GQL_COMMAND);

gqlFunction({
  variables: inputParams
})
```



# App Loading

We load as much as user-specific information up-front when app is being loaded once authenticated. The data fetch is completely based on the user email address that passed the authentication.


## FULL DATA FETCH (mobile app)

Mobile app is fetching all the info that is relevant. See web version below for web MVP.

This query is limited to the specific user by automatically matching <code>user_account.uid</code> to idToken in authenticated jwt token.

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


Parameter | Type | Description
--------- | ----------- | -----------
email     | text | ID from baby_profile table
start_at  | timestamp | data load look back date. E.g. if entered '2024-11-01', it fetches meal plan from '2024-11-01'.


> Example input parameter:

```graphql
{
  "email": "email@email.com",
  "start_at": "2024-11-01"
}
```


## MVP DATA FETCH (web)

This query is limited to the specific user by automatically matching <code>user_account.uid</code> to idToken in authenticated jwt token.


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


Parameter | Type | Description
--------- | ----------- | -----------
email     | text | user's email
start_at  | timestamp | data load look back date. E.g. if entered '2024-11-01', it fetches meal plan from '2024-11-01'.


<aside class="notice">
Note that it is pulling recipes that includes this user's recipes and saved recipes. And you need to parse whether the recipe is this user's own or someone elses (e.g. saved) manually based on the author's ID.
</aside>


> Example input parameter:

```graphql
{
  "email": "email@email.com",
  "start_at": "2024-11-01"
}
```


# Account


## Insert Account

AWS Cognito authenticates users, but we keep track of their data separately. Thus, when user creates an account through Cognito, we need to make sure that it gets inserted into our <code>user_account</code> data table. And because the user doesn't have a record in our database yet, we allow the row to be inserted without restriction.

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

Parameter | Type        | Description
--------- | ----------- | -----------
email     | text        | the user's email
uid       | uuid        | uid provided by Cognito
nickname  | text        | the user's nickname. It's the first part from email address by default in mobile app logic
thumbnailLink | text    | S3 key pointing to this user's thumbnail.


## Reactivate Account
This is called when the uid from Cognito doesn't match with the uid from our database. This is necessary because otherwise, user token is not authorized.

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

This part is usually created during onboarding process.

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


## Read User Profile

Reading user profile should be processed when app gets loaded.



## Update Account Profile

TODO. Diet restriction and food allergy should be updated here in the same query in future.

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




# Recipes

Make sure to have these filters:
- removed_at is null
- recipe_status_id == COMPLETE
- user_account.user_status is either ACTIVE_FREE or ACTIVE_PREMIUM


## Get all recipes




## Search recipes by search term

Searching recipes based on title, description, recipe author's nickname, ingredients and tags.

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

<aside class="notice">
We track what terms are being searched. Make sure to save search.
</aside>

```graphql
mutation SaveSearch($user_account_id: uuid!, $search_term: String!) {
  insert_search_record(objects: {user_account_id: $user_account_id, search_term: $search_term}){
    affected_rows
  }
}
```


## Fetch recipes for a specific ingredient

This functionality is used primarily for food tracking page. 

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


## Fetch recipes by one tag

This functionality can be used when there is one specific recipe tag that you want to fetch recipes for. On the main page, when a user picks one category among popular recipe categories, this is the query to use.

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

## Fetch recipes by more than one tags

This functionality can be used when there is one or more specific recipe tags that you want to fetch. The main feed page on mobile app uses this query.

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

## Fetch recipe detail by slug

Every recipe has a slug. This query fetches recipe detail given the slug.

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

## Save favorite recipe

```graphql
mutation SaveFavoriteRecipe($user_account_id: uuid!, $recipe_id: uuid!) {
  insert_user_saved_recipe_one(object: {user_account_id: $user_account_id, recipe_id: $recipe_id}){
    id
  }
}

```


## Create a new recipe
<code>
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
</code>

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

## Edit a recipe
<code>
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
</code>

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

# ------ BOOKMARK

# Meal Plans

import { gql } from '@apollo/client';

export const GQL_SAVE_MEAL_PLAN = gql`
    mutation AddToMealPlan(
    $planned_at: timestamp!, 
    $recipe_id: uuid!, 
    $user_account_id: uuid!) {
    insert_recipe_hist(objects: {
        user_account_id: $user_account_id, 
        recipe_id: $recipe_id, 
        planned_at: $planned_at}){
            returning {
            id
            }
        }
    }
`

export const GQL_REMOVE_MEAL_PLAN = gql`
mutation RemoveMealPlan($recipe_hist_id: uuid!) {
  update_recipe_hist(where: {id: {_eq: $recipe_hist_id}}, _set: {removed_at: now}){
    returning{
      id
    }
  }
}
`

export const GQL_UPDATE_MEAL_PLAN = gql`
mutation UpdateMealPlan($recipe_hist_id: uuid!, $planned_at: timestamp!) {
  update_recipe_hist(where: {id: {_eq: $recipe_hist_id}}, _set: {planned_at: $planned_at}){
    returning{
      id
    }
  }
}
`

export const GQL_GET_MEAL_PLAN = gql`
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
`

export const GQL_SAVE_REACTION = gql`
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
            }) {
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
`

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
export const GQL_EARN_MILESTONE = gql`
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

`


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

export const GQL_ADD_ALL_MEALS = gql`
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

`




# Social


import { gql } from "@apollo/client";

{
  "commenter_uid": "CqznSvrThLfZjrDqlDQrFVhDwpI2",
  "post_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
  "comment":"hi"
}


export const GQL_COMMENT_RECIPE = gql`
mutation InsertComment($commenter_id: uuid!, $recipe_id: uuid!, $comment: String!) {
  insert_comments_one(object: {commenter_id: $commenter_id, recipe_id: $recipe_id, comment: $comment}) {
    id
  }
}
`


{
  "post_id": "7177a1bb-875f-4836-939a-7600e7c134cd",
  "liker_uid":"CqznSvrThLfZjrDqlDQrFVhDwpI2"
}

export const GQL_LIKE_RECIPE = gql`
mutation InsertLike($liker_id: uuid!, $recipe_id: uuid!) {
  insert_likes_one(object: {liker_id: $liker_id, recipe_id: $recipe_id}){
    id
  }
}
`


export const GQL_UNLIKE_RECIPE = gql`
mutation RemoveLike ($liker_id: uuid!, $recipe_id:uuid!){
  update_likes(where: {liker_id: {_eq: $liker_id}, recipe_id: {_eq: $recipe_id}}, 
    _set: {removed_at: now}){
    returning {
      id
    }
  }
}
`




{
    "recipe_offset": 0,
    "recipe_limit": 10,
    "uid": "test"
  }
export const GQL_GET_FEED_FOLLOWING = gql`
query GetFeed(
  $recipe_offset: Int!, 
  $recipe_limit: Int!, 
  $user_account_id: uuid!) @cached {
  recipe(
    offset: $recipe_offset, 
    limit: $recipe_limit, 
    order_by: {created_at: desc},
    where: {
      recipe_status_id: {_eq: COMPLETE}, 
      user_account: {_or: [
        {uid: {_ilike: "%admin%"}},
        {id: {_eq: $user_account_id}},
        {user_follows: {follower_id: {_eq: $user_account_id}}}]}}) {
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
`
   {user_follows: {follower_id: {_neq: $user_account_id}}} TODO put this back to filter out the accounts that user is already following


export const GQL_UNFOLLOW = gql`
mutation UpdateUnfollow($follower_id: uuid!, $followee_id: uuid!) {
  update_user_follow(where: {followee_id: {_eq: $followee_id}, follower_id: {_eq: $follower_id}}, _set: {removed_at: now}){
    returning {
      id
    }
  }
}
`


{
  "follower_uid": "CqznSvrThLfZjrDqlDQrFVhDwpI2",
  "followee_uid": "C83AqbDPRzdhPghTdIGRt6tl2Do2"
}

export const GQL_FOLLOW = gql`
mutation InsertFollow ($follower_id: uuid!, $followee_id: uuid!) {
  insert_user_follow_one(object: {follower_id: $follower_id, followee_id: $followee_id}) {
    id
  }
}
`


{
  "uid": "test",
  "recipe_offset": 0,
  "recipe_limit": 10,
}

export const GQL_GET_USER = gql`
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
    recipes{
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


`




{
  "tag_filter": ["ALLERGEN_FREE_EGG", "STAGE_ONE"]
}
export const GQL_GET_FEED_FOR_YOU = gql`
query MyQuery($tag_filter:[tag_enum!]) {
  recipe(where: {recipe_tags: {tag_id: {_in: $tag_filter,}}}) {
    id
    link_pic
  }
}
`



export const GQL_INSERT_RECIPE_FLAG = gql`
mutation ReportRecipe($reporter_id: uuid = "", $recipe_id: uuid = "") {
  insert_flag(objects: {reporter_id: $reporter_id, recipe_id: $recipe_id}) {
    returning{
    reporter_id      
    }
  }
}
 
`

export const GQL_INSERT_USER_FLAG = gql`
mutation ReportUser($reporter_id: uuid = "", $user_id: uuid = "") {
  insert_flag(objects: {reporter_id: $reporter_id, user_id: $user_id}) {
    returning{
    reporter_id      
    }
  }
}

 
`



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

Parameter | Type | Description
--------- | ----------- | -----------
baby_id | uuid | ID from baby_profile table
days | number | the baby's age in days
growth_type_id | enum | 'WEIGHT', 'HEIGHT', or 'BMI'
percentile | integer | the growth percentile that the app has calculated
value | float | the growth measure for weight, height, or bmi



### Read

This part is automatically pulled in when app is loaded with proper account information


### Update



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

### Read


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




# Codes

## Recipe Tag

## Diet Restriction

## Food Allerty




`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```


> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>
