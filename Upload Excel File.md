# Upload Excel File

PHP script that handles Excel file uploads. This will include file validation, processing, and storage.

```php
<?php
// PHP Excel Upload and Processing Script

// Required libraries - you'll need to install these via Composer
// composer require phpoffice/phpspreadsheet

// Include Composer autoloader
require 'vendor/autoload.php';

use PhpOffice\PhpSpreadsheet\IOFactory;
use PhpOffice\PhpSpreadsheet\Spreadsheet;

// Configuration
$uploadDir = 'uploads/';
$allowedExtensions = ['xlsx', 'xls', 'csv'];
$maxFileSize = 5 * 1024 * 1024; // 5MB

// Create uploads directory if it doesn't exist
if (!file_exists($uploadDir)) {
    mkdir($uploadDir, 0777, true);
}

// Process the form submission
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $result = [
        'success' => false,
        'message' => '',
        'data' => []
    ];
    
    // Check if file was uploaded
    if (!isset($_FILES['excelFile']) || $_FILES['excelFile']['error'] !== UPLOAD_ERR_OK) {
        $result['message'] = 'File upload failed: ' . getUploadErrorMessage($_FILES['excelFile']['error']);
    } else {
        $file = $_FILES['excelFile'];
        
        // Validate file size
        if ($file['size'] > $maxFileSize) {
            $result['message'] = 'File is too large. Maximum size is ' . ($maxFileSize / 1024 / 1024) . 'MB';
        } else {
            // Validate file extension
            $fileExtension = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
            if (!in_array($fileExtension, $allowedExtensions)) {
                $result['message'] = 'Invalid file format. Allowed formats: ' . implode(', ', $allowedExtensions);
            } else {
                // Generate unique filename
                $newFilename = uniqid() . '.' . $fileExtension;
                $uploadFilePath = $uploadDir . $newFilename;
                
                // Move the file to upload directory
                if (move_uploaded_file($file['tmp_name'], $uploadFilePath)) {
                    // Process Excel file
                    try {
                        $spreadsheet = IOFactory::load($uploadFilePath);
                        $worksheet = $spreadsheet->getActiveSheet();
                        $rows = $worksheet->toArray();
                        
                        // Get headers from first row
                        $headers = array_shift($rows);
                        
                        // Process data
                        $processedData = [];
                        foreach ($rows as $row) {
                            $rowData = [];
                            foreach ($headers as $index => $header) {
                                $rowData[$header] = $row[$index] ?? '';
                            }
                            $processedData[] = $rowData;
                        }
                        
                        // Here you can save the data to database or perform other operations
                        // For example, connecting to MySQL and inserting the data
                        
                        $result['success'] = true;
                        $result['message'] = 'File uploaded and processed successfully';
                        $result['data'] = [
                            'filename' => $newFilename,
                            'rows' => count($processedData),
                            'headers' => $headers,
                            'preview' => array_slice($processedData, 0, 5) // Preview first 5 rows
                        ];
                        
                        // Optionally store data in session for further processing
                        $_SESSION['excel_data'] = $processedData;
                        $_SESSION['excel_headers'] = $headers;
                        
                    } catch (Exception $e) {
                        $result['message'] = 'Error processing file: ' . $e->getMessage();
                        // Delete the uploaded file if processing fails
                        unlink($uploadFilePath);
                    }
                } else {
                    $result['message'] = 'Failed to save the uploaded file';
                }
            }
        }
    }
    
    // Return JSON response if Ajax request
    if (isset($_SERVER['HTTP_X_REQUESTED_WITH']) && $_SERVER['HTTP_X_REQUESTED_WITH'] === 'XMLHttpRequest') {
        header('Content-Type: application/json');
        echo json_encode($result);
        exit;
    }
}

/**
 * Get human-readable upload error message
 */
function getUploadErrorMessage($errorCode) {
    switch ($errorCode) {
        case UPLOAD_ERR_INI_SIZE:
            return 'The uploaded file exceeds the upload_max_filesize directive in php.ini';
        case UPLOAD_ERR_FORM_SIZE:
            return 'The uploaded file exceeds the MAX_FILE_SIZE directive in the HTML form';
        case UPLOAD_ERR_PARTIAL:
            return 'The uploaded file was only partially uploaded';
        case UPLOAD_ERR_NO_FILE:
            return 'No file was uploaded';
        case UPLOAD_ERR_NO_TMP_DIR:
            return 'Missing a temporary folder';
        case UPLOAD_ERR_CANT_WRITE:
            return 'Failed to write file to disk';
        case UPLOAD_ERR_EXTENSION:
            return 'A PHP extension stopped the file upload';
        default:
            return 'Unknown upload error';
    }
}
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Excel Upload</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        .alert {
            padding: 15px;
            margin-bottom: 20px;
            border-radius: 4px;
        }
        .alert-success {
            background-color: #dff0d8;
            color: #3c763d;
        }
        .alert-danger {
            background-color: #f2dede;
            color: #a94442;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        table, th, td {
            border: 1px solid #ddd;
        }
        th, td {
            padding: 8px;
            text-align: left;
        }
        th {
            background-color: #f2f2f2;
        }
    </style>
</head>
<body>
    <h1>Excel File Upload</h1>
    
    <?php if ($_SERVER['REQUEST_METHOD'] === 'POST'): ?>
        <?php if (isset($result)): ?>
            <div class="alert <?php echo $result['success'] ? 'alert-success' : 'alert-danger'; ?>">
                <?php echo $result['message']; ?>
            </div>
            
            <?php if ($result['success'] && !empty($result['data']['preview'])): ?>
                <h2>Data Preview</h2>
                <p>Total rows: <?php echo $result['data']['rows']; ?></p>
                
                <table>
                    <thead>
                        <tr>
                            <?php foreach ($result['data']['headers'] as $header): ?>
                                <th><?php echo htmlspecialchars($header); ?></th>
                            <?php endforeach; ?>
                        </tr>
                    </thead>
                    <tbody>
                        <?php foreach ($result['data']['preview'] as $row): ?>
                            <tr>
                                <?php foreach ($result['data']['headers'] as $header): ?>
                                    <td><?php echo htmlspecialchars($row[$header]); ?></td>
                                <?php endforeach; ?>
                            </tr>
                        <?php endforeach; ?>
                    </tbody>
                </table>
                
                <p>
                    <a href="process.php">Process this data</a> | 
                    <a href="<?php echo $_SERVER['PHP_SELF']; ?>">Upload another file</a>
                </p>
            <?php endif; ?>
        <?php endif; ?>
    <?php endif; ?>
    
    <form action="<?php echo $_SERVER['PHP_SELF']; ?>" method="post" enctype="multipart/form-data">
        <div class="form-group">
            <label for="excelFile">Select Excel File:</label>
            <input type="file" name="excelFile" id="excelFile" required>
            <small>Allowed formats: <?php echo implode(', ', $allowedExtensions); ?> (Max: <?php echo $maxFileSize / 1024 / 1024; ?>MB)</small>
        </div>
        
        <div class="form-group">
            <button type="submit">Upload and Process</button>
        </div>
    </form>
</body>
</html>

```

This PHP script provides a complete solution for uploading and processing Excel files. Here's what it includes:

1. **File Validation**:
   - Validates file extensions (.xlsx, .xls, .csv)
   - Checks file size (default 5MB limit)
   - Validates upload integrity

2. **Excel Processing**:
   - Uses PHPSpreadsheet library to read Excel data
   - Extracts headers from the first row
   - Processes all rows into an associative array

3. **User Interface**:
   - Clean HTML form for file uploads
   - Success/error messaging
   - Data preview table for the first 5 rows
   - Simple styling included

4. **Security Features**:
   - Sanitizes output with htmlspecialchars()
   - Generates unique filenames to prevent overwriting
   - Provides detailed error messages

To use this script, you'll need to:

1. Install the required PHPSpreadsheet library using Composer:
   ```
   composer require phpoffice/phpspreadsheet
   ```

2. Create a directory for file uploads (the script will attempt to create it if it doesn't exist)

3. Optionally, create a "process.php" file to handle the actual database operations or other data processing

You can easily extend this script to connect to a database and insert the data, or to provide more advanced validation and processing features.

Would you like me to explain any particular part of the code in more detail?