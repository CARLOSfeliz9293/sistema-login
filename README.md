# sistema-login
<?php
// install.php - Instalador del sistema

// Verificar si ya está instalado
if(file_exists('config/database.php')) {
    die('El sistema ya está instalado. Elimina el archivo config/database.php para reinstalar.');
}

// Procesar el formulario de instalación
if($_SERVER['REQUEST_METHOD'] === 'POST') {
    $db_host = $_POST['db_host'] ?? 'localhost';
    $db_name = $_POST['db_name'];
    $db_user = $_POST['db_user'];
    $db_pass = $_POST['db_pass'];
    
    try {
        // Intentar conexión
        $pdo = new PDO("mysql:host=$db_host", $db_user, $db_pass);
        $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        
        // Crear la base de datos si no existe
        $pdo->exec("CREATE DATABASE IF NOT EXISTS `$db_name`");
        $pdo->exec("USE `$db_name`");
        
        // Crear tabla de usuarios
        $pdo->exec("CREATE TABLE IF NOT EXISTS `users` (
            `id` INT AUTO_INCREMENT PRIMARY KEY,
            `username` VARCHAR(50) NOT NULL UNIQUE,
            `password` VARCHAR(255) NOT NULL,
            `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        )");
        
        // Insertar usuario por defecto
        $default_user = 'demo';
        $default_pass = password_hash('tareafacil25', PASSWORD_DEFAULT);
        
        $stmt = $pdo->prepare("INSERT INTO `users` (`username`, `password`) VALUES (?, ?)");
        $stmt->execute([$default_user, $default_pass]);
        
        // Crear archivo de configuración
        $config_content = "<?php
define('DB_HOST', '$db_host');
define('DB_NAME', '$db_name');
define('DB_USER', '$db_user');
define('DB_PASS', '$db_pass');
";
        
        // Crear directorio config si no existe
        if(!is_dir('config')) {
            mkdir('config', 0755, true);
        }
        
        file_put_contents('config/database.php', $config_content);
        
        // Redirigir al login
        header('Location: login.php');
        exit;
        
    } catch(PDOException $e) {
        $error = "Error de conexión: " . $e->getMessage();
    }
}
?>

<!DOCTYPE html>
<html>
<head>
    <title>Instalación del Sistema</title>
    <style>
        body { font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto; padding: 20px; }
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; }
        input { width: 100%; padding: 8px; box-sizing: border-box; }
        button { background: #4CAF50; color: white; padding: 10px 15px; border: none; cursor: pointer; }
        .error { color: red; }
    </style>
</head>
<body>
    <h1>Instalación del Sistema</h1>
    
    <?php if(isset($error)): ?>
        <div class="error"><?= htmlspecialchars($error) ?></div>
    <?php endif; ?>
    
    <form method="post">
        <div class="form-group">
            <label for="db_host">Servidor de Base de Datos:</label>
            <input type="text" id="db_host" name="db_host" value="localhost" required>
        </div>
        
        <div class="form-group">
            <label for="db_name">Nombre de la Base de Datos:</label>
            <input type="text" id="db_name" name="db_name" required>
        </div>
        
        <div class="form-group">
            <label for="db_user">Usuario de MySQL:</label>
            <input type="text" id="db_user" name="db_user" required>
        </div>
        
        <div class="form-group">
            <label for="db_pass">Contraseña de MySQL:</label>
            <input type="password" id="db_pass" name="db_pass">
        </div>
        
        <button type="submit">Instalar</button>
    </form>
    
    <p>Después de la instalación, se creará un usuario por defecto:<br>
    <strong>Usuario:</strong> demo<br>
    <strong>Contraseña:</strong> tareafacil25</p>
</body>
</html>
