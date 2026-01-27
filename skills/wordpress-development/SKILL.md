---
alwaysApply: true
name: wordpress-dev
description: Wordpress development guidelines
---
As a Senior Software Engineer and UX/UI Designer with over two decades of experience, I have synthesized your requirements into a single, comprehensive set of WordPress development rules. This unified document merges the technical rigor of both source files to ensure high-quality, secure, and maintainable code.

---

## WordPress Development Rules

### 1. Core Principles & Architecture
* **Role**: Expert in WordPress, PHP, and modern web development.
* **Response Style**: Provide concise, technical responses with accurate PHP examples.
* **OOP Focus**: Emphasize Object-Oriented Programming (OOP) for modularity and scalability.
* **DRY Principle**: Focus on code reusability through iteration and modularization; avoid duplication.
* **Standard Compliance**: Strictly adhere to WordPress PHP Coding Standards and best practices.

### 2. PHP Development Standards

* **Modern PHP**: Utilize PHP 7.4+ features (e.g., typed properties, arrow functions).
* **Strict Typing**: Enable strict typing by adding `declare(strict_types=1);` at the top of PHP files.
* **Naming Conventions**: Use descriptive function, variable, and file names.
* **Directory Structure**: Use lowercase with hyphens (e.g., `wp-content/themes/my-theme`).
* **Documentation**: Add clear, descriptive comments to improve maintainability.

### 3. WordPress Core Integration

* **Hooks System**: Always use actions and filters to extend functionality; never modify core files.
* **Core APIs**: Leverage built-in WordPress functions and APIs whenever available.
* **Database (wpdb)**: Use the `$wpdb` abstraction layer for custom database interactions.
* Always apply `prepare()` statements to prevent SQL injection.
* Manage schema changes using the `dbDelta()` function.
* **Asset Management**: Use `wp_enqueue_script()` and `wp_enqueue_style()` for all scripts and styles.

### 4. Security & Data Handling
* **Validation & Sanitization**: Use built-in WordPress functions for all data validation, sanitization, and escaping.
* **Nonce Verification**: Implement proper nonce verification for all form submissions and AJAX requests.
* **Auth & Permissions**: Use WordPress's built-in user roles, capabilities, and authentication mechanisms for access control.


### 5. Error Handling & Performance

* **Error Logging**: Utilize WordPress debug logging (`WP_DEBUG_LOG`) and create custom handlers when necessary.
* **Exceptions**: Use try-catch blocks for expected or controlled exception handling.
* **Caching**: Apply the Transients API for data caching and optimization.
* **Background Tasks**: Schedule long-running or automated tasks using the `wp_cron()` API.

### 6. Theme & Plugin Development

* **Theme Structure**: Follow the WordPress template hierarchy and utilize `functions.php`.
* **Template Tags**: Use template tags and conditional tags for dynamic content handling.
* **Extensibility**: Follow the Plugin API to ensure modular and scalable extensions.
* **Custom Content**: Implement custom post types and taxonomies to extend core functionality.
* **Internationalization**: Use WordPress i18n functions for localization.
* **Dependencies**: Use Composer for dependency management in advanced projects.
* **Testing**: Write unit tests using the `WP_UnitTestCase` framework.

### 7. Communication & API

* **AJAX/REST**: Handle backend requests via `admin-ajax.php` or the WordPress REST API.
* **Configuration**: Store configuration and settings using the Options API.
* **UX/UI Utilities**: Implement proper pagination using functions like `paginate_links()`.