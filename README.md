# shibudb-client-php
ShibuDb client library for PHP

# ShibuDb PHP Client

A comprehensive PHP client for ShibuDb database that supports authentication, key-value operations, vector similarity search, space management, and connection pooling. Compatible with **PHP 5.6+**.

## Features

- 🔐 **Authentication & User Management**: Secure login with role-based access control
- 🔑 **Key-Value Operations**: Traditional key-value storage with PUT, GET, DELETE operations
- 🧮 **Vector Similarity Search**: Advanced vector operations with multiple index types
- 🗂️ **Space Management**: Create, delete, and manage different storage spaces
- 🛡️ **Error Handling**: Comprehensive error handling with custom exceptions
- 📊 **Connection Management**: Automatic connection handling with destructors
- 🔗 **Connection Pooling**: High-performance connection pooling for concurrent operations

## Installation

### Prerequisites

1. **ShibuDb Server**: Ensure the ShibuDb server is running
   ```bash
   # Start the server (requires sudo)
   sudo shibudb start 4444
   ```

2. **PHP Requirements**: 
   - PHP 5.6 or higher
   - No external PHP extensions required (uses built-in `stream_socket_client`, `json_encode`, `json_decode`)

### Setup

#### Composer (Recommended)

```bash
composer require shibudb-org/shibudb-client-php
```

Then in your PHP code:

```php
<?php
require_once 'vendor/autoload.php';

$client = new ShibuDbClient('localhost', 4444);
$client->authenticate('admin', 'admin');
```

#### Manual Installation

1. **Copy the client file to your project**:
   ```bash
   cp ShibuDbClient.php /path/to/your/project/
   ```

2. **Include the client**:
   ```php
   require_once '/path/to/ShibuDbClient.php';
   ```

## Quick Start

### Basic Connection and Authentication

```php
<?php
require_once 'ShibuDbClient.php';

// Create client and authenticate
$client = new ShibuDbClient('localhost', 4444);
$client->authenticate('admin', 'admin');

// Your operations here
$response = $client->listSpaces();
print_r($response);

// Close connection (or let destructor handle it)
$client->close();
```

### Convenience Function

```php
<?php
// Using the convenience function
$client = shibudb_connect('localhost', 4444, 'admin', 'admin');
$response = $client->listSpaces();
$client->close();
```

### Connection Pooling

```php
<?php
require_once 'ShibuDbClient.php';

// Create a connection pool
$pool = new ShibuDbConnectionPool(
    'localhost', 4444,        // host, port
    'admin', 'admin',         // username, password
    30, 2, 10, 30             // timeout, min_size, max_size, acquire_timeout
);

// Use pooled connections
$client = $pool->getConnection();
$response = $client->listSpaces();
print_r($response);
$pool->releaseConnection($client);

// Get pool statistics
$stats = $pool->getStats();
print_r($stats);

// Close pool
$pool->close();
```

### Key-Value Operations

```php
<?php
// Create and use a space
$client->createSpace('mytable', 'key-value');
$client->useSpace('mytable');

// Basic operations
$client->put('name', 'John Doe');
$response = $client->get('name');
echo $response['value'];  // "John Doe"

$client->delete('name');
```

### Vector Operations

```php
<?php
// Create a vector space
$client->createSpace('vectors', 'vector', 128, 'Flat', 'L2');
$client->useSpace('vectors');

// Insert vectors
$client->insertVector(1, array(0.1, 0.2, 0.3, /* ... */));
$client->insertVector(2, array(0.4, 0.5, 0.6, /* ... */));

// Search for similar vectors
$results = $client->searchTopk(array(0.1, 0.2, 0.3, /* ... */), 5);
echo $results['message'];  // Search results

// Range search
$results = $client->rangeSearch(array(0.1, 0.2, 0.3, /* ... */), 0.5);
```

## API Reference

### ShibuDbClient

#### Constructor
```php
new ShibuDbClient($host = 'localhost', $port = 4444, $timeout = 30)
```

#### Authentication
```php
$client->authenticate($username, $password) -> array
```

#### Space Management
```php
$client->createSpace($name, $engineType = 'key-value', $dimension = null, 
                    $indexType = 'Flat', $metric = 'L2') -> array
$client->deleteSpace($name) -> array
$client->listSpaces() -> array
$client->useSpace($name) -> array
```

#### Key-Value Operations
```php
$client->put($key, $value, $space = null) -> array
$client->get($key, $space = null) -> array
$client->delete($key, $space = null) -> array
```

#### Vector Operations
```php
$client->insertVector($vectorId, $vector, $space = null) -> array
$client->searchTopk($queryVector, $k = 1, $space = null) -> array
$client->rangeSearch($queryVector, $radius, $space = null) -> array
$client->getVector($vectorId, $space = null) -> array
```

#### User Management (Admin Only)
```php
$client->createUser($user) -> array
$client->updateUserPassword($username, $newPassword) -> array
$client->updateUserRole($username, $newRole) -> array
$client->updateUserPermissions($username, $permissions) -> array
$client->deleteUser($username) -> array
$client->getUser($username) -> array
```

### Data Models

#### User Array
```php
$user = array(
    'username' => 'user1',
    'password' => 'password123',
    'role' => 'user',  // 'admin' or 'user'
    'permissions' => array(
        'mytable' => 'read',
        'vectortable' => 'write'
    )
);
```

### Exceptions

- `ShibuDbException`: Base exception for all client errors
- `ShibuDbAuthenticationError`: Raised when authentication fails
- `ShibuDbConnectionError`: Raised when connection fails
- `ShibuDbQueryError`: Raised when query execution fails
- `ShibuDbPoolExhaustedError`: Raised when connection pool is exhausted

## Examples

### Complete Example

```php
<?php
require_once 'ShibuDbClient.php';

function main() {
    try {
        // Connect and authenticate
        $client = new ShibuDbClient('localhost', 4444);
        $client->authenticate('admin', 'admin');
        
        // Create spaces
        $client->createSpace('users', 'key-value');
        $client->createSpace('embeddings', 'vector', 128);
        
        // Store user data
        $client->useSpace('users');
        $client->put('user1', 'Alice Johnson');
        $client->put('user2', 'Bob Smith');
        
        // Store embeddings
        $client->useSpace('embeddings');
        $vector1 = array_fill(0, 128, 0.1);  // Example vector
        $vector2 = array_fill(0, 128, 0.2);  // Example vector
        $client->insertVector(1, $vector1);
        $client->insertVector(2, $vector2);
        
        // Search for similar embeddings
        $queryVector = array_fill(0, 128, 0.1);
        $results = $client->searchTopk($queryVector, 5);
        echo "Search results: " . print_r($results, true);
        
        $client->close();
        
    } catch (ShibuDbException $e) {
        echo "Error: " . $e->getMessage() . "\n";
    }
}

main();
```

### Error Handling

```php
<?php
require_once 'ShibuDbClient.php';

try {
    $client = new ShibuDbClient('localhost', 4444);
    $client->authenticate('admin', 'admin');
    
    // Your operations here
    
} catch (ShibuDbAuthenticationError $e) {
    echo "Authentication failed: " . $e->getMessage() . "\n";
} catch (ShibuDbConnectionError $e) {
    echo "Connection failed: " . $e->getMessage() . "\n";
} catch (ShibuDbQueryError $e) {
    echo "Query failed: " . $e->getMessage() . "\n";
} finally {
    if (isset($client)) {
        $client->close();
    }
}
```

### Advanced Usage

```php
<?php
require_once 'ShibuDbClient.php';

// Create admin user
$adminUser = array(
    'username' => 'admin',
    'password' => 'adminpass',
    'role' => 'admin'
);

// Create regular user with permissions
$user = array(
    'username' => 'user1',
    'password' => 'userpass',
    'role' => 'user',
    'permissions' => array(
        'mytable' => 'read',
        'vectortable' => 'write'
    )
);

$client = new ShibuDbClient('localhost', 4444);
$client->authenticate('admin', 'admin');

// Create users
$client->createUser($user);

// Create spaces for different purposes
$client->createSpace('users', 'key-value');
$client->createSpace('products', 'key-value');
$client->createSpace('embeddings', 'vector', 256);
$client->createSpace('recommendations', 'vector', 512);

// Store data in different spaces
$client->useSpace('users');
$client->put('user1', 'Alice Johnson');

$client->useSpace('embeddings');
$queryVector = array_fill(0, 256, 0.1);
$client->insertVector(1, $queryVector);

// Search for recommendations
$results = $client->searchTopk($queryVector, 10);
print_r($results);

$client->close();
```

## Running Examples

### Simple Test
```bash
php simple_test.php
```

### Comprehensive Examples
```bash
php example.php
```

### Connection Pooling Examples
```bash
php pool_test.php
```

### Comprehensive Connection Pooling Tests
```bash
php comprehensive_pool_test.php
```

## Connection Pooling

The ShibuDb client supports connection pooling for high-performance concurrent operations. Connection pooling provides:

- **Connection Reuse**: Efficiently reuse database connections
- **Concurrent Operations**: Support for multiple simultaneous operations
- **Automatic Health Checks**: Health monitoring of connections
- **Configurable Pool Size**: Adjustable minimum and maximum pool sizes
- **Timeout Handling**: Configurable connection acquisition timeouts

### Pool Configuration

```php
<?php
require_once 'ShibuDbClient.php';

// Create pool with custom configuration
$pool = new ShibuDbConnectionPool(
    'localhost', 4444,        // host, port
    'admin', 'admin',         // username, password
    30,                       // timeout (seconds)
    2,                        // min_size (minimum connections in pool)
    10,                       // max_size (maximum connections in pool)
    30                        // acquire_timeout (timeout for acquiring connection)
);
```

### Using Connection Pools

```php
<?php
// Basic usage
$client = $pool->getConnection();
$response = $client->listSpaces();
print_r($response);
$pool->releaseConnection($client);

// Concurrent operations (example with multiple requests)
$clients = array();
for ($i = 0; $i < 5; $i++) {
    $client = $pool->getConnection();
    $client->createSpace("space_$i", 'key-value');
    $client->useSpace("space_$i");
    $client->put("key_$i", "value_$i");
    $response = $client->get("key_$i");
    print_r($response);
    $pool->releaseConnection($client);
}
```

### Pool Statistics

```php
<?php
// Get pool statistics
$stats = $pool->getStats();
echo "Pool size: " . $stats['pool_size'] . "\n";
echo "Active connections: " . $stats['active_connections'] . "\n";
echo "Min size: " . $stats['min_size'] . "\n";
echo "Max size: " . $stats['max_size'] . "\n";
```

### Error Handling with Pools

```php
<?php
require_once 'ShibuDbClient.php';

try {
    $client = $pool->getConnection();
    // Your operations here
    $pool->releaseConnection($client);
    
} catch (ShibuDbPoolExhaustedError $e) {
    echo "Pool exhausted: " . $e->getMessage() . "\n";
} catch (ShibuDbAuthenticationError $e) {
    echo "Authentication failed: " . $e->getMessage() . "\n";
} catch (ShibuDbConnectionError $e) {
    echo "Connection failed: " . $e->getMessage() . "\n";
} finally {
    if (isset($pool)) {
        $pool->close();
    }
}
```

## Engine Types

### Key-Value Engine
- Traditional key-value storage
- Supports PUT, GET, DELETE operations
- No dimension required

### Vector Engine
- Vector similarity search
- Multiple index types:
    - **Flat**: Exact search (default)
    - **HNSW**: Hierarchical Navigable Small World
    - **IVF**: Inverted File Index
    - **IVF with PQ**: Product Quantization
- Distance metrics:
    - **L2**: Euclidean distance (default)
    - **IP**: Inner product
    - **COS**: Cosine similarity

## Security

- **Authentication Required**: All operations require valid credentials
- **Role-Based Access**: Admin and user roles with different permissions
- **Space-Level Permissions**: Read/write permissions per space
- **Connection Security**: TCP-based communication with timeout handling

## PHP 5.6 Compatibility

This client is designed for PHP 5.6+ compatibility:

- No scalar type hints (PHP 7+ feature)
- No return type declarations (PHP 7+ feature)
- No null coalescing operator (`??`) (PHP 7+ feature)
- Uses `array()` syntax for clarity
- Compatible with `json_decode()` strictness in PHP 5.6
- Uses built-in PHP functions only (no external dependencies)

## Troubleshooting

### Common Issues

1. **Connection Failed**
    - Ensure ShibuDb server is running: `sudo shibudb start 4444`
    - Check server port and host settings
    - Verify firewall settings
    - Check PHP error logs for connection errors

2. **Authentication Failed**
    - Verify username and password
    - Ensure user exists in the system
    - Check user permissions

3. **Space Not Found**
    - Use `listSpaces()` to see available spaces
    - Create space before using: `createSpace()`
    - Use `useSpace()` to switch to a space

4. **Vector Dimension Mismatch**
    - Ensure vector dimension matches space dimension
    - Check space creation parameters
    - Verify vector format (array of floats)

5. **JSON Parsing Errors**
    - Check PHP version compatibility (PHP 5.6+)
    - Verify JSON response format from server
    - Check for special characters in data

### Debug Mode

Enable PHP error reporting:
```php
<?php
error_reporting(E_ALL);
ini_set('display_errors', 1);

// Your ShibuDb code here
```

## Running Tests

Tests mirror the Python client test structure:

| Test File | Description | Server Required |
|-----------|-------------|-----------------|
| `simple_test.php` | Basic key-value and vector operations | Yes |
| `pool_test.php` | Connection pooling basics | Yes |
| `comprehensive_pool_test.php` | Full pool tests (stats, configs, errors) | Yes |
| `json_parsing_test.php` | JSON parsing with special chars, fallback | No |
| `special_chars_test.php` | Special character handling, encoding | No |

```bash
# Tests that require ShibuDb server (sudo shibudb start 4444)
php simple_test.php
php pool_test.php
php comprehensive_pool_test.php

# Tests that run without server
php json_parsing_test.php
php special_chars_test.php
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Add tests for new functionality
4. Ensure PHP 5.6+ compatibility
5. Submit a pull request

## License

This client is provided as-is for use with ShibuDb database.
