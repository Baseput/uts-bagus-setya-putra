# uts-bagus-setya-putra
repository untuk uts web service
1. Nama: Bagus Setya Putra
2. NIM: 21 01 65 0002
3. Deskripsi: Saya membuat integrasi web service dengan studi kasus manajemen video game PS dengan menngunakan postman
berikut source code diantaranya:

**File index.PHP**
```
<?php
error_reporting(E_ALL);
ini_set('display_errors', 1);

header("Content-Type: application/json; charset=UTF-8");
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");
header("Access-Control-Allow-Headers: Content-Type, Access-Control-Allow-Headers, Authorization, X-Requested-With");

require_once 'db.php';

$database = new Database();
$db = $database->getConnection();

$method = $_SERVER['REQUEST_METHOD'];

$request = $_SERVER['REQUEST_URI'];
$script_name = dirname($_SERVER['SCRIPT_NAME']);
$path = substr(parse_url($request, PHP_URL_PATH), strlen($script_name));
$pathFragments = explode('/', trim($path, '/'));
$resource = isset($pathFragments[0]) ? $pathFragments[0] : null;
$id = isset($pathFragments[1]) ? (int)$pathFragments[1] : null;

function sendResponse($data, $status_code = 200) {
    http_response_code($status_code);
    echo json_encode($data);
    exit();
}

function getInput() {
    return json_decode(file_get_contents('php://input'), true);
}

if ($resource === 'api' && isset($pathFragments[1]) && $pathFragments[1] === 'books') {
    switch ($method) {
        case 'GET':
            if (isset($pathFragments[4])) {
                $stmt = $db->prepare("SELECT * FROM books WHERE id = :id");
                $stmt->bindParam(':id', $id, PDO::PARAM_INT);
                $stmt->execute();
                $book = $stmt->fetch(PDO::FETCH_ASSOC);
                if ($book) {
                    sendResponse($book);
                } else {
                    sendResponse(['message' => 'Data tidak ditemukan'], 404);
                }
            } else {
                $stmt = $db->prepare("SELECT * FROM books");
                $stmt->execute();
                $books = $stmt->fetchAll(PDO::FETCH_ASSOC);
                sendResponse($books);
            }
            break;

        case 'POST':
            $input = getInput();
            if (isset($input['unit_number']) && isset($input['type'])) {
                $stmt = $db->prepare("INSERT INTO books (unit_number, type) VALUES (:unit_number, :type)");
                $stmt->bindParam(':unit_number', $input['unit_number']);
                $stmt->bindParam(':type', $input['type']);
                if ($stmt->execute()) {
                    $new_id = $db->lastInsertId();
                    $new_book = [
                        'id' => (int)$new_id,
                        'unit_number' => $input['unit_number'],
                        'type' => $input['type']
                    ];
                    sendResponse($new_book, 201);
                } else {
                    sendResponse(['message' => 'Gagal menambahkan data'], 500);
                }
            } else {
                sendResponse(['message' => 'Data tidak lengkap'], 400);
            }
            break;

        case 'PUT':
            if (isset($pathFragments[4])) {
                // Memperbarui buku berdasarkan ID
                $input = getInput();
                // Cek apakah buku ada
                $stmt = $db->prepare("SELECT * FROM books WHERE id = :id");
                $stmt->bindParam(':id', $id, PDO::PARAM_INT);
                $stmt->execute();
                $book = $stmt->fetch(PDO::FETCH_ASSOC);
                if ($book) {
                    // Update data
                    $title = isset($input['unit_number']) ? $input['unit_number'] : $book['unit_number'];
                    $author = isset($input['type']) ? $input['type'] : $book['type'];
                    $stmt = $db->prepare("UPDATE books SET unit_number = :unit_number, type = :type WHERE id = :id");
                    $stmt->bindParam(':unit_number', $unit_number);
                    $stmt->bindParam(':type', $type);
                    $stmt->bindParam(':id', $id, PDO::PARAM_INT);
                    if ($stmt->execute()) {
                        $updated_book = [
                            'id' => (int)$id,
                            'title' => $unit_number,
                            'author' => $type
                        ];
                        sendResponse($updated_book);
                    } else {
                        sendResponse(['message' => 'Gagal memperbarui data'], 500);
                    }
                } else {
                    sendResponse(['message' => 'Data tidak ditemukan'], 404);
                }
            } else {
                sendResponse(['message' => 'ID tidak disediakan'], 400);
            }
            break;

        case 'DELETE':
            if (isset($pathFragments[4])) {
                $stmt = $db->prepare("DELETE FROM books WHERE id = :id");
                $stmt->bindParam(':id', $id, PDO::PARAM_INT);
                if ($stmt->execute()) {
                    if ($stmt->rowCount() > 0) {
                        sendResponse(['message' => 'Data dihapus']);
                    } else {
                        sendResponse(['message' => 'Data tidak ditemukan'], 404);
                    }
                } else {
                    sendResponse(['message' => 'Gagal menghapus Data'], 500);
                }
            } else {
                sendResponse(['message' => 'ID tidak disediakan'], 400);
            }
            break;

        default:
            sendResponse(['message' => 'Metode tidak diizinkan'], 405);
    }
} else {
    sendResponse(['message' => 'Endpoint tidak ditemukan'], 404);
}
?>
```
