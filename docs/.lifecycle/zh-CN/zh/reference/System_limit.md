---
displayed_sidebar: "Chinese"
---

# System Restrictions

This article introduces the points to note when using the StarRocks system.

- StarRocks uses the MySQL protocol for communication. Users can connect to the StarRocks cluster using MySQL Client or JDBC. When choosing the MySQL Client version, it is recommended to use version 5.1 and later. Versions before 5.1 do not support usernames with more than 16 characters.

- Requirements for data directory names (Catalog), database names, table names, view names, partition names, column names, usernames, and role names:
   - Can only consist of digits (0-9), letters (a-z or A-Z), and underscores (\_). **Usernames can be named using pure numbers.**
   - Cannot exceed 64 characters. **Among these, data directory names, database names, table names, and column names cannot exceed 1023 characters (≤ 1023).**
   - Data directory names, database names, table names, view names, partition names, and role names can only start with lowercase or uppercase letters.
   - Column names can start with an underscore.
   - Data directory names, database names, table names, view names, usernames, and role names are **case-sensitive**, while column names and partition names are **case-insensitive**.

- Requirements for label names:
   When importing data, you can specify labels for tasks. Label names can consist of digits (0-9), uppercase and lowercase letters (a-z or A-Z), and underscores (\_), and cannot exceed 128 characters. There is no requirement on the starting character for label names.
- When creating a table, the Key column cannot use FLOAT or DOUBLE types, and DECIMAL type can be used to represent decimals.
- Maximum length for VARCHAR:
   - For versions before StarRocks 2.1, the length range is 1~65533 bytes.
   - 【In public beta】Starting from version 2.1 of StarRocks, the length range is 1~1048576 bytes. 1048578 (maximum row value) - 2 (length identifier, recording the actual data length) = 1048576.
- StarRocks only supports UTF8 encoding and does not support other encodings such as GB.
- StarRocks does not support modifying column names in a table.
- By default, the maximum number of subqueries in a single query is 10000.