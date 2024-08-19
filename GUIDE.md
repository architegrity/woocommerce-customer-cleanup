# Guide to Removing Orphaned WooCommerce Customers from WordPress

## Overview
When running a WooCommerce store, it's possible to accumulate unwanted or bot-created customer entries. This guide will walk you through the process of removing these entries from your WooCommerce store and cleaning up related data from the database. Following this guide will help ensure your WooCommerce customer data remains clean and accurate.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Step 1: Backup Your Database](#step-1-backup-your-database)
3. [Step 2: Identify and Delete Suspicious Users](#step-2-identify-and-delete-suspicious-users)
4. [Step 3: Clean Up Orphaned Usermeta Data](#step-3-clean-up-orphaned-usermeta-data)
5. [Step 4: Remove Orphaned Data from WooCommerce Tables](#step-4-remove-orphaned-data-from-woocommerce-tables)
   - [Step 4.1: Remove from `wc_customer_lookup`](#step-41-remove-from-wc_customer_lookup)
   - [Step 4.2: Remove from `wc_order_stats`](#step-42-remove-from-wc_order_stats)
6. [Step 5: Clear WooCommerce Analytics Cache](#step-5-clear-woocommerce-analytics-cache)
7. [Step 6: Regenerate the Customers List](#step-6-regenerate-the-customers-list)
8. [Step 7: Verify Cleanup](#step-7-verify-cleanup)
9. [Conclusion](#conclusion)

## Prerequisites
- Access to your WordPress admin dashboard.
- Access to your database via phpMyAdmin or a similar tool.
- A recent backup of your WordPress database.

## Important Note on Table Prefixes
The SQL queries provided in this guide use `dQ8RX_` as a placeholder for your actual WordPress table prefix. WordPress table prefixes are often set during installation and might differ on your site. You must replace `dQ8RX_` with your actual table prefix in all SQL queries.

### How to Find Your Table Prefix
1. Open your WordPress installation's `wp-config.php` file.
2. Look for the line that defines the `$table_prefix` variable.
3. Use the value of `$table_prefix` in place of `dQ8RX_` in the SQL queries provided.

Example:
If your table prefix is `wp_`, replace `dQ8RX_users` with `wp_users`.

## Step 1: Backup Your Database
Before making any changes, always create a full backup of your WordPress database to ensure you can restore your site if something goes wrong.

## Step 2: Identify and Delete Suspicious Users
1. Log in to your WordPress Admin Dashboard.
2. Navigate to **WooCommerce > Customers** and identify suspicious customer entries (e.g., missing names, strange email addresses).
3. Use the following SQL query to delete users who have `NULL` or empty `first_name` and `last_name` values in the `usermeta` table:

    ```sql
    DELETE FROM dQ8RX_users
    WHERE ID IN (
        SELECT u.ID
        FROM dQ8RX_users u
        LEFT JOIN dQ8RX_usermeta m1 ON u.ID = m1.user_id AND m1.meta_key = 'first_name'
        LEFT JOIN dQ8RX_usermeta m2 ON u.ID = m2.user_id AND m2.meta_key = 'last_name'
        WHERE (m1.meta_value IS NULL OR m1.meta_value = '')
        AND (m2.meta_value IS NULL OR m2.meta_value = '')
    );
    ```

## Step 3: Clean Up Orphaned Usermeta Data
After deleting the users, remove any orphaned records in the `usermeta` table that no longer have corresponding users:

```sql
DELETE FROM dQ8RX_usermeta
WHERE user_id NOT IN (
    SELECT ID FROM dQ8RX_users
);
```

## Step 4: Remove Orphaned Data from WooCommerce Tables
WooCommerce stores customer data in its own tables for analytics and reporting. Clean up these tables by removing entries related to deleted users:

### Step 4.1: Remove from wc_customer_lookup:

```sql
DELETE FROM dQ8RX_wc_customer_lookup
WHERE user_id NOT IN (
    SELECT ID FROM dQ8RX_users
);
```

### Step 4.2: Remove from wc_order_stats:

```sql
DELETE FROM dQ8RX_wc_order_stats
WHERE customer_id NOT IN (
    SELECT ID FROM dQ8RX_users
);
```
## Step 5: Clear WooCommerce Analytics Cache
To ensure that WooCommerce accurately reflects the current state of your data:
1. Purge Transients:
- Use a plugin like "WP-Optimize" to clear WooCommerce transients.
2. Regenerate Analytics:
- Navigate to WooCommerce > Status > Tools.
- Use the "Clear analytics cache" and "Rebuild reports" options to regenerate WooCommerce analytics data.

## Step 6: Regenerate the Customers List
- Go to WooCommerce > Status > Tools.
- Use the "Regenerate customers" tool to rebuild the customers list in WooCommerce.

## Step 7: Verify Cleanup
Return to WooCommerce > Customers and verify that the unwanted or bot-created customer entries have been successfully removed.

## Conclusion
By following these steps, you can clean up your WooCommerce customer data, ensuring that only legitimate customers are stored in your database. This process removes orphaned entries from various WooCommerce tables and ensures that your customer data is accurate and up-to-date.

Important Reminder: Always ensure you have a full database backup before performing any bulk deletions or database modifications. This will protect your site from accidental data loss.