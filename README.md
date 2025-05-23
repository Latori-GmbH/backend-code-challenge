# Backend Code Challenge

## Objective
This challenge is designed to assess your ability to integrate with the Shopify Admin API, manage product data, and handle order webhooks. You will be building a small Ruby on Rails application that interacts with a Shopify store.

## Resources
* **Shopify Store URL:** `https://latori-teststore.myshopify.com/admin`
* **Shopify Store Login email:** `Shared with the candidate`
* **Shopify Store Login password:** `Shared with the candidate`
* **Shopify API Key:** `Shared with the candidate`
* **Shopify API Secret Key:** `Shared with the candidate`
* **Shopify API Access Token (Password):** `Shared with the candidate`
* **Product Export CSV File:** [products.csv](data/import/products/products.csv)
* **Order Estimated Delivery Date Metafield Import JSON Format:** [estimated_delivery_{id}.json](data/export/orders/estimated_delivery_{id}.json)

## Shopify API Documentation

* **Authentication & Setup:**
  * **`shopify_api` Gem:** [GitHub Repository](https://github.com/Shopify/shopify-api-ruby)
* **GraphQL Admin API:**
  * **GraphQL:** [Shopify Docs](https://shopify.dev/docs/apps/build/graphql)
  * **GraphQL Admin API Introduction:** [Shopify Docs](https://shopify.dev/docs/api/admin-graphql)
  * **Product Mutations:**
    * `productSet`: [Shopify Docs](https://shopify.dev/docs/api/admin-graphql/latest/mutations/productSet)
  * **Metafield Mutations:**
    * `metafieldSet`: [Shopify Docs](https://shopify.dev/docs/api/admin-graphql/latest/mutations/metafieldsSet)
  * **File Upload API:**
    * **Manage media for products (Guide):** [Shopify Docs](https://shopify.dev/docs/apps/build/online-store/product-media)
    * `stagedUploadsCreate`: [Shopify Docs](https://shopify.dev/docs/api/admin-graphql/latest/mutations/stagedUploadsCreate)
    * `fileCreate`: [Shopify Docs](https://shopify.dev/docs/api/admin-graphql/latest/mutations/fileCreate)
  * **Webhooks:**
    * **Webhooks Introduction:** [Shopify Docs](https://shopify.dev/docs/apps/build/webhooks) 
    * `webhookSubscriptionCreate`: [Shopify Docs](https://shopify.dev/docs/api/admin-graphql/latest/mutations/webhookSubscriptionCreate)

## Requirements
### 1. Product Synchronization

**User Story**:
As a store administrator,
I want to synchronize product data from a CSV file into my Shopify store,
So that my product catalog is up-to-date with correct information, including estimated delivery dates for each product as metafields.

*Acceptance Criteria*:
Given the application has valid Shopify API credentials,
And a `products.csv` file is available,
When a Rake task / Scheduled task (e.g., `rails sync:products`) is executed,
Then the system authenticates with Shopify using the `shopify_api` gem.

*Acceptance Criteria*:
Given the `sync:products` task is running,
When it processes a row from the `products.csv` where the product does not exist in Shopify,
Then a new product and its associated variants are created in Shopify using the GraphQL Admin API,
And the `Estimated Delivery Date in Days` column from the CSV is saved as a product metafield with the following specifications:
* Namespace: `HANDLE_OF_YOUR_NAME_IN_SNAKE_CASE` e.g. `max_mustermann`
* Key: `estimated_delivery`
* Type: `number_integer`

*Acceptance Criteria*:
Given the `sync:products` task is running,
When it processes a row from the `products.csv` where the product already exists in Shopify,
Then the existing product's information (title, description, vendor, etc.) and its variants' information (price, sku, etc.) are updated using the GraphQL Admin API,
And the `Estimated Delivery Date in Days` from the CSV is updated as a product metafield (`max_mustermann.estimated_delivery` of type `number_integer`).

*Acceptance Criteria*:
Given the `sync:products` task is running,
When the synchronization process completes,
Then clear output is logged to the console, indicating the number of products created, updated, and any errors encountered.

### 2. Order Synchronization

**User Story**:
As a store administrator,
When a new order is marked as paid in Shopify,
I want a JSON file containing the order information (name, line items etc.) including the estimated delivery date for each line item to be automatically generated and attached to the order as a metafield,
So that e.g. fulfillment services can easily access this information.

*Acceptance Criteria*:
Given the application has valid Shopify API credentials,
Then the application should subscribes to the Shopify `orders/paid` webhook topic using the GraphQL Admin API.

*Acceptance Criteria*:
Given the application is subscribed to the `orders/paid` webhook,
And an endpoint is configured to receive these webhooks,
When Shopify sends an `orders/paid` webhook notification,
Then the application validates the integrity and origin (HMAC) of the incoming webhook and proceeds to process its data

*Acceptance Criteria*:
Given an `orders/paid` webhook is received,
When the application processes the order's line items,
Then for each line item, it retrieves the `estimated_delivery` from the corresponding product's metafield (`max_mustermann.estimated_delivery`).

*Acceptance Criteria*:
Given the estimated delivery dates for all line items in a paid order have been retrieved,
When the application prepares the data for upload,
Then a JSON file is generated in memory that strictly follows the format specified in `estimated_delivery_{id}.json`, where `{id}` is the Shopify Order ID.

*Acceptance Criteria*:
Given a JSON file with estimated delivery dates has been generated for a paid order,
Then the application uploads the JSON file permanenty to Shopify's file system.

*Acceptance Criteria*:
Given the JSON file has been successfully uploaded and a file [GID](https://shopify.dev/docs/api/usage/gids) is obtained,
Then a new metafield of type `file_reference` is created and attached to the order,
And this metafield has the previously defined namespace, key `delivery_dates`, and its value is the [GID](https://shopify.dev/docs/api/usage/gids) of the uploaded file.

*Acceptance Criteria*:
Given an `orders/paid` webhook is being processed,
When a product in the order does not have the `max_mustermann.estimated_delivery` metafield,
Then the application logs this event,
And continues processing the rest of the order without failing.

## Submission Guidelines

* Please submit your code as a GitHub repository (private, inviting us as a collaborator) or a ZIP file.
* Ensure your code is well-documented and includes instructions for running the application.
* Ensure your code is clean, well-structured, and follows best practices.
* Include a README file with an overview of your implementation, design choices, any assumptions made, and any additional notes.
* We'd like you to build this project on a modern Ruby on Rails stack. While we expect latest versions of the framework and its dependencies, the specific choice of architecture, design patterns, and additional tools is yours.
* Consider the requirements our definition of "what" needs to be done. We are most interested in seeing "how" you solve it. We encourage you to use any libraries or techniques you feel best showcase your skills.
* If you have any questions or need clarification, feel free to reach out.
