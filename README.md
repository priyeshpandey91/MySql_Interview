
# Practical Database Design for an E-Commerce Site

Let's assume we are building a basic e-commerce platform and want to design a database that includes a catalog of products, their details, and the categories to which they belong. We also want to design tables for users, orders, and other essential components.

Here are the basic tables and their structure:

### 1. **Users Table**
Stores information about users who visit or make purchases on the site.

```sql
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL,
    password VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);


### 2. **Categories Table**
Contains information about product categories.

```sql
CREATE TABLE categories (
    category_id INT AUTO_INCREMENT PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL,
    description TEXT
);
```

### 3. **Products Table**
Contains information about products. This will be the primary catalog table.

```sql
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    stock_quantity INT NOT NULL,
    category_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES categories(category_id)
);
```

### 4. **Orders Table**
Contains orders made by users.

```sql
CREATE TABLE orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total_amount DECIMAL(10, 2) NOT NULL,
    status ENUM('pending', 'completed', 'canceled') DEFAULT 'pending',
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

### 5. **Order_Items Table**
Contains information about the products in each order.

```sql
CREATE TABLE order_items (
    order_item_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

### 6. **Product_Images Table**
Stores product images to be displayed on the site.

```sql
CREATE TABLE product_images (
    image_id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT,
    image_url VARCHAR(255) NOT NULL,
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

---

### Real-World Catalog and Product Data Fetching Query (Joining Tables)

Now, we’ll write a query to fetch and group products from the catalog along with their category details, filtering by product price and categorizing by the category they belong to. 

#### Objective:
1. Fetch all products from the catalog.
2. Join with the categories table to get the category name.
3. Group by category and list products under each category.
4. Show product details like name, price, and stock quantity.
5. Limit the result to products priced above 50.

```sql
SELECT 
    c.category_name,
    p.product_name,
    p.price,
    p.stock_quantity,
    GROUP_CONCAT(pi.image_url) AS product_images
FROM 
    products p
JOIN 
    categories c ON p.category_id = c.category_id
LEFT JOIN 
    product_images pi ON p.product_id = pi.product_id
WHERE 
    p.price > 50
GROUP BY 
    c.category_name, p.product_name
ORDER BY 
    c.category_name, p.product_name;
```

### Explanation of the Query:
1. **SELECT**: The query selects `category_name` from the `categories` table, and `product_name`, `price`, `stock_quantity` from the `products` table. It also fetches a concatenated list of image URLs from the `product_images` table.
2. **JOIN**: The `products` table is joined with the `categories` table using the `category_id` to get the category name. The `product_images` table is joined with `products` to fetch associated images.
3. **WHERE**: Filters out products that have a price lower than or equal to 50.
4. **GROUP_CONCAT**: Aggregates multiple image URLs for a product into a single field.
5. **GROUP BY**: Groups the data by category and product, allowing us to see all products listed under their respective categories.
6. **ORDER BY**: Sorts the results by category and product name.

---

### Sample Data for `categories`, `products`, and `product_images`

#### `categories` Table
```sql
INSERT INTO categories (category_name, description) VALUES
('Electronics', 'Gadgets and electronic devices'),
('Fashion', 'Clothing, shoes, and accessories'),
('Home & Kitchen', 'Furniture and kitchen appliances');
```

#### `products` Table
```sql
INSERT INTO products (product_name, description, price, stock_quantity, category_id) VALUES
('Smartphone', 'Latest model with great features', 600.00, 50, 1),
('Laptop', 'High performance laptop', 1200.00, 30, 1),
('T-shirt', 'Comfortable cotton t-shirt', 25.00, 100, 2),
('Blender', 'Powerful blender for smoothies', 80.00, 20, 3),
('Washing Machine', 'High-efficiency washing machine', 400.00, 10, 3);
```

#### `product_images` Table
```sql
INSERT INTO product_images (product_id, image_url) VALUES
(1, 'images/smartphone1.jpg'),
(1, 'images/smartphone2.jpg'),
(2, 'images/laptop1.jpg'),
(3, 'images/tshirt1.jpg'),
(4, 'images/blender1.jpg'),
(5, 'images/washing_machine1.jpg');
```

---

### Sample Output:

Assuming you run the SQL query, the output could look like:

| category_name  | product_name | price | stock_quantity | product_images                                      |
|----------------|--------------|-------|----------------|-----------------------------------------------------|
| Electronics    | Smartphone   | 600.00| 50             | images/smartphone1.jpg,images/smartphone2.jpg        |
| Electronics    | Laptop       | 1200.00| 30            | images/laptop1.jpg                                  |
| Home & Kitchen | Blender      | 80.00 | 20             | images/blender1.jpg                                 |
| Home & Kitchen | Washing Machine | 400.00 | 10         | images/washing_machine1.jpg                         |

---

This query is a common use case in e-commerce sites where product catalog data needs to be retrieved and displayed with filtering, grouping, and aggregation.
```
