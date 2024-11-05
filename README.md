# uts-bagus-setya-putra
repository untuk uts web service

File PHP
source code php dibuat dengan nama ps_api.php
<?php
header("Content-Type: application/json; charset=UTF-8");
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");
header("Access-Control-Allow-Headers: Content-Type, Access-Control-Allow-Headers, Authorization, X-Requested-With");

$method = $_SERVER['REQUEST_METHOD'];
$request = [];

if (isset($_SERVER['PATH_INFO'])) {
    $request = explode('/', trim($_SERVER['PATH_INFO'],'/'));
}

function getConnection() {
    $host = 'localhost';
    $db   = 'playstation_units';
    $user = 'root';
    $pass = ''; // Ganti dengan password MySQL Anda jika ada
    $charset = 'utf8mb4';

    $dsn = "mysql:host=$host;dbname=$db;charset=$charset";
    $options = [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false,
    ];
    try {
        return new PDO($dsn, $user, $pass, $options);
    } catch (\PDOException $e) {
        throw new \PDOException($e->getMessage(), (int)$e->getCode());
    }
}

function response($status, $data = NULL) {
    header("HTTP/1.1 " . $status);
    if ($data) {
        echo json_encode($data);
    }
    exit();
}

$db = getConnection();

switch ($method) {
    case 'GET':
        if (!empty($request) && isset($request[0])) {
            $id = $request[0];
            $stmt = $db->prepare("SELECT * FROM ps WHERE id = ?");
            $stmt->execute([$id]);
            $ps = $stmt->fetch();
            if ($ps) {
                response(200, $ps);
            } else {
                response(404, ["message" => "data not found"]);
            }
        } else {
            $stmt = $db->query("SELECT * FROM ps");
            $books = $stmt->fetchAll();
            response(200, $ps);
        }
        break;
    
    case 'POST':
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->title) || !isset($data->author) || !isset($data->year)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "INSERT INTO ps (unit_number, type, status_barang, hourly_rate) VALUES (?, ?, ?, ?)";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->unit_number, $data->type, $data->status_barang, $data->hourly_rate,])) {
            response(201, ["message" => "data dibuat", "id" => $db->lastInsertId()]);
        } else {
            response(500, ["message" => "gagal menambahkan"]);
        }
        break;
    
    case 'PUT':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "data ID is required"]);
        }
        $id = $request[0];
        $data = json_decode(file_get_contents("php://input"));
        if (!isset($data->unit_number) || !isset($data->type)  || !isset($data->status_barang) || !isset($data-> hourly_rate)) {
            response(400, ["message" => "Missing required fields"]);
        }
        $sql = "UPDATE ps SET unit_number = ?, type = ?, status_barang = ?, hourly_rate = ? WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$data->unit_number, $data->type, $data->status_barang, $data->hourly_rate, $id])) {
            response(200, ["message" => "data telah di tambahkan"]);
        } else {
            response(500, ["message" => "gagal menambahkan"]);
        }
        break;
    
    case 'DELETE':
        if (empty($request) || !isset($request[0])) {
            response(400, ["message" => "data ID is required"]);
        }
        $id = $request[0];
        $sql = "DELETE FROM ps WHERE id = ?";
        $stmt = $db->prepare($sql);
        if ($stmt->execute([$id])) {
            response(200, ["message" => "data dihapus"]);
        } else {
            response(500, ["message" => "gagal menghapus data"]);
        }
        break;
    
    default:
        response(405, ["message" => "Method not allowed"]);
        break;
}
?>

Script SQL
CREATE TABLE ps (
    id INT AUTO_INCREMENT PRIMARY KEY,
    unit_number VARCHAR(10) NOT NULL,
    type VARCHAR(20) NOT NULL,
    status_barang VARCHAR(30) NOT NULL,
    hourly_rate VARCHAR(10) NOT NULL);

INSERT INTO ps (id, unit_number, type, status_barang, hourly_rate) VALUES
('01', '11', 'ps 3', 'lengkap', '1 jam' ),
('02', '12', 'ps 4', 'lengkap', '2 jam'),
('03', '13', 'ps 3', 'lengkap', '1 jam'),
('04', '14', 'ps 3', '1 stick', '3 jam'),
('05', '15', 'ps 4', '1 memory', '1 jam');

