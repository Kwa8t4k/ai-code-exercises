# Exercise 5: API Documentation

## API Endpoint Selected
JavaScript/Express — Product API
Endpoint: GET /api/products (List Products with filtering and pagination)

---

## Original Endpoint Code
````javascript
productRouter.get('/', async (req, res) => {
  try {
    const {
      category, minPrice, maxPrice,
      sort = 'createdAt', order = 'desc',
      page = 1, limit = 20, inStock
    } = req.query;

    const filter = {};
    if (category) filter.category = category;
    if (minPrice !== undefined || maxPrice !== undefined) {
      filter.price = {};
      if (minPrice !== undefined) filter.price.$gte = parseFloat(minPrice);
      if (maxPrice !== undefined) filter.price.$lte = parseFloat(maxPrice);
    }
    if (inStock === 'true') filter.stockQuantity = { $gt: 0 };

    const skip = (parseInt(page) - 1) * parseInt(limit);
    const sortOptions = {};
    sortOptions[sort] = order === 'asc' ? 1 : -1;

    const products = await ProductModel.find(filter)
      .sort(sortOptions).skip(skip).limit(parseInt(limit));

    const totalProducts = await ProductModel.countDocuments(filter);

    return res.status(200).json({
      products,
      pagination: {
        total: totalProducts,
        page: parseInt(page),
        limit: parseInt(limit),
        pages: Math.ceil(totalProducts / parseInt(limit))
      }
    });
  } catch (error) {
    return res.status(500).json({
      error: 'Server error',
      message: 'Failed to fetch products'
    });
  }
});
````

---

## Prompt 1 Output: Comprehensive Endpoint Documentation

### GET /api/products — List Products

**Purpose:**
Retrieves a paginated list of products from the database. Supports flexible filtering
by category, price range, and stock availability, along with customisable sorting
and pagination controls.

**Authentication:** None required. This is a public endpoint.

**Base URL:** `https://api.example.com`

---

#### Query Parameters

| Parameter | Type    | Required | Default    | Description |
|-----------|---------|----------|------------|-------------|
| category  | String  | No       | —          | Filter by product category (exact match) |
| minPrice  | Number  | No       | —          | Minimum price (inclusive). Filters products with price >= this value |
| maxPrice  | Number  | No       | —          | Maximum price (inclusive). Filters products with price <= this value |
| sort      | String  | No       | createdAt  | Field to sort results by (e.g. price, name, createdAt) |
| order     | String  | No       | desc       | Sort direction. Accepted values: asc, desc |
| page      | Integer | No       | 1          | Page number for pagination. Must be >= 1 |
| limit     | Integer | No       | 20         | Number of results per page. Maximum 100 |
| inStock   | Boolean | No       | —          | When true, only returns products with stockQuantity > 0 |

---

#### Success Response

**Status:** 200 OK
````json
{
  "products": [
    {
      "_id": "61fa9bcf5c130b2e6d675432",
      "name": "Wireless Headphones",
      "description": "High-quality wireless headphones with noise cancellation",
      "price": 89.99,
      "category": "electronics",
      "stockQuantity": 45,
      "createdAt": "2024-02-01T15:32:47Z",
      "updatedAt": "2024-03-15T09:21:08Z"
    }
  ],
  "pagination": {
    "total": 42,
    "page": 1,
    "limit": 20,
    "pages": 3
  }
}
````

---

#### Error Responses

| Status Code | Error | Message | Cause |
|-------------|-------|---------|-------|
| 500 | Server error | Failed to fetch products | Database connection failure or unexpected server error |

---

#### Example 1: Get all electronics in stock, sorted by price

**Request:**
````
GET /api/products?category=electronics&inStock=true&sort=price&order=asc
````

**Response (200 OK):**
````json
{
  "products": [
    {
      "_id": "61fa9bcf5c130b2e6d675435",
      "name": "Bluetooth Speaker",
      "price": 49.99,
      "category": "electronics",
      "stockQuantity": 32
    },
    {
      "_id": "61fa9bcf5c130b2e6d675432",
      "name": "Wireless Headphones",
      "price": 89.99,
      "category": "electronics",
      "stockQuantity": 45
    }
  ],
  "pagination": {
    "total": 2,
    "page": 1,
    "limit": 20,
    "pages": 1
  }
}
````

---

#### Example 2: Get second page of products priced between R20 and R100

**Request:**
````
GET /api/products?minPrice=20&maxPrice=100&page=2&limit=10
````

**Response (200 OK):**
````json
{
  "products": [
    {
      "_id": "61fa9bcf5c130b2e6d675439",
      "name": "T-Shirt",
      "price": 24.99,
      "category": "clothing",
      "stockQuantity": 150
    }
  ],
  "pagination": {
    "total": 18,
    "page": 2,
    "limit": 10,
    "pages": 2
  }
}
````

---

#### Special Considerations
- Product IDs are MongoDB ObjectId strings (24 hex characters)
- All timestamps are in ISO 8601 format (UTC)
- The maximum allowed limit per page is 100 items
- If both minPrice and maxPrice are provided, results fall within that range (inclusive on both ends)
- The sort field must match a real field name on the Product model, otherwise results may be unpredictable

---

## Prompt 2 Output: OpenAPI/Swagger Format
````yaml
openapi: 3.0.0
info:
  title: Products API
  description: API for retrieving and filtering product listings
  version: 1.0.0
servers:
  - url: https://api.example.com
paths:
  /api/products:
    get:
      summary: List products
      description: >
        Returns a paginated list of products. Supports filtering by category,
        price range, and stock availability. Results can be sorted by any
        product field in ascending or descending order.
      parameters:
        - in: query
          name: category
          schema:
            type: string
          description: Filter products by category (exact match)
        - in: query
          name: minPrice
          schema:
            type: number
            format: float
          description: Minimum price filter (inclusive)
        - in: query
          name: maxPrice
          schema:
            type: number
            format: float
          description: Maximum price filter (inclusive)
        - in: query
          name: sort
          schema:
            type: string
            default: createdAt
          description: Field name to sort results by
        - in: query
          name: order
          schema:
            type: string
            enum: [asc, desc]
            default: desc
          description: Sort direction
        - in: query
          name: page
          schema:
            type: integer
            minimum: 1
            default: 1
          description: Page number for pagination
        - in: query
          name: limit
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
          description: Number of items per page
        - in: query
          name: inStock
          schema:
            type: boolean
          description: When true, only return products with stock greater than 0
      responses:
        '200':
          description: Successful response with product list and pagination
          content:
            application/json:
              schema:
                type: object
                properties:
                  products:
                    type: array
                    items:
                      $ref: '#/components/schemas/Product'
                  pagination:
                    $ref: '#/components/schemas/Pagination'
        '500':
          description: Internal server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
components:
  schemas:
    Product:
      type: object
      properties:
        _id:
          type: string
          description: MongoDB ObjectId (24 hex characters)
        name:
          type: string
        description:
          type: string
        price:
          type: number
          format: float
        category:
          type: string
        stockQuantity:
          type: integer
        createdAt:
          type: string
          format: date-time
        updatedAt:
          type: string
          format: date-time
    Pagination:
      type: object
      properties:
        total:
          type: integer
          description: Total number of matching products
        page:
          type: integer
          description: Current page number
        limit:
          type: integer
          description: Items per page
        pages:
          type: integer
          description: Total number of pages
    Error:
      type: object
      properties:
        error:
          type: string
        message:
          type: string
````

---

## Prompt 3 Output: Developer Usage Guide

### Getting Started with the Products API

**Target audience:** Junior to intermediate developers integrating with the API for the first time.

---

#### Authentication
This endpoint requires no authentication. You can call it directly without any headers or tokens.

---

#### Making Your First Request

Using JavaScript fetch:
````javascript
const response = await fetch('https://api.example.com/api/products');
const data = await response.json();
console.log(data.products);     // Array of product objects
console.log(data.pagination);   // Pagination metadata
````

Using curl:
````bash
curl https://api.example.com/api/products
````

---

#### Filtering Products

Filter by category:
````javascript
const response = await fetch(
  'https://api.example.com/api/products?category=electronics'
);
````

Filter by price range:
````javascript
const response = await fetch(
  'https://api.example.com/api/products?minPrice=20&maxPrice=100'
);
````

Only show items in stock:
````javascript
const response = await fetch(
  'https://api.example.com/api/products?inStock=true'
);
````

Combine multiple filters:
````javascript
const response = await fetch(
  'https://api.example.com/api/products?category=electronics&inStock=true&maxPrice=100'
);
````

---

#### Handling Pagination

Always check pagination before assuming you have all results:
````javascript
const response = await fetch('https://api.example.com/api/products?page=1&limit=10');
const data = await response.json();

console.log(`Showing page ${data.pagination.page} of ${data.pagination.pages}`);
console.log(`Total products: ${data.pagination.total}`);

// Load next page if available
if (data.pagination.page < data.pagination.pages) {
  const nextPage = await fetch(
    `https://api.example.com/api/products?page=${data.pagination.page + 1}&limit=10`
  );
}
````

---

#### Handling Errors
````javascript
const response = await fetch('https://api.example.com/api/products');

if (!response.ok) {
  const error = await response.json();
  console.error(`Error ${response.status}: ${error.message}`);
  // Handle or retry as appropriate
  return;
}

const data = await response.json();
````

---

#### Common Mistakes to Avoid
- Do not set limit above 100 — the API will not enforce this but results may be unpredictable
- The inStock parameter must be the string "true" not a boolean — it comes from a query string
- Always handle the 500 error case — database errors can occur unexpectedly

---

## Reflection

### Most challenging part to document
The pagination response structure needed careful documentation because
it returns metadata alongside the data array — a pattern that confuses
developers expecting just an array back.

### How I adjusted my prompts
For Prompt 2, I specified "output as valid YAML OpenAPI 3.0 format"
to avoid getting a partial or incorrectly structured document.

### Which format was most effective
The Markdown format (Prompt 1) was most readable for human developers.
The OpenAPI YAML (Prompt 2) was most useful for tooling like Swagger UI.
Both serve different purposes and ideally both would exist in a real project.