1. Сервер (TCP Server)
Сервер будет хранить ники подключённых клиентов и перенаправлять сообщения.


Код сервера (server.py):

import socket
from threading import Thread

# Настройки сервера
SERVER_HOST = "0.0.0.0"
SERVER_PORT = 5002
clients = {}  # Словарь: ник -> сокет клиента

# Создание TCP-сокета
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server_socket.bind((SERVER_HOST, SERVER_PORT))
server_socket.listen(5)
print(f"[*] Сервер слушает: {SERVER_HOST}:{SERVER_PORT}")

def handle_client(client_socket, client_address):
    try:
        # Получение ника от клиента
        nickname = client_socket.recv(1024).decode().strip()
        
        # Проверка уникальности ника
        if nickname in clients:
            client_socket.send("Ник уже занят!\n".encode())
            client_socket.close()
            return
        clients[nickname] = client_socket
        print(f"[+] {nickname} подключился.")
        client_socket.send("Ник принят.\n".encode())

        # Обработка сообщений
        while True:
            message = client_socket.recv(1024).decode()
            if not message:
                break
            parts = message.split(" ", 2)
            if len(parts) == 3 and parts == "SEND":
                target_nick, msg = parts[1](https://thepythoncode.com/article/make-a-chat-room-application-in-python), parts[2](https://metanit.com/sharp/net/4.4.php)
                if target_nick in clients:
                    clients[target_nick].send(f"От {nickname}: {msg}\n".encode())
                else:
                    client_socket.send(f"Ошибка: ник '{target_nick}' не найден!\n".encode())
    except Exception as e:
        print(f"[!] Ошибка: {e}")
    finally:
        nickname = [k for k, v in clients.items() if v == client_socket]
        del clients[nickname]
        client_socket.close()
        print(f"[-] {nickname} отключился.")

# Прослушивание подключений
while True:
    client_sock, client_addr = server_socket.accept()
    print(f"[*] Подключение от {client_addr}")
    client_handler = Thread(target=handle_client, args=(client_sock, client_addr))
    client_handler.start()


2. Клиент (TCP Client)
Клиент подключается к серверу, отправляет ник и обменивается сообщениями.

Код клиента (client.py):

import socket

SERVER_HOST = "127.0.0.1"  # Замените на IP сервера
SERVER_PORT = 5002

# Подключение к серверу
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client_socket.connect((SERVER_HOST, SERVER_PORT))

# Ввод ника
nickname = input("Введите ваш ник: ").strip()
client_socket.send(nickname.encode())

# Проверка ответа сервера
response = client_socket.recv(1024).decode()
if "принят" not in response:
    print(response)
    client_socket.close()
    exit()

print(response)

# Основной цикл обмена сообщениями
try:
    while True:
        message = input("> ").strip()
        if not message:
            continue
        parts = message.split(" ", 1)
        if len(parts) == 2:
            target_nick, msg = parts, parts[1](https://thepythoncode.com/article/make-a-chat-room-application-in-python)
            client_socket.send(f"SEND {target_nick} {msg}".encode())
        else:
            print("Формат: [ник получателя] [сообщение]")
finally:
    client_socket.close()
