# Authenticated-SQL-Injection-Time-based-Blind-in-Product-Expiry-Alert-Management-System
A critical **SQL Injection** vulnerability was discovered in the `edit-admin.php` component of the "Product Expiry Alert Management System". The application allows authenticated users to edit profiles but fails to properly sanitize the `id` GET parameter before using it in a SQL query.
