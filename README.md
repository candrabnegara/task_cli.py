# task_cli.py
Project 1 
import json
import os
import sys
from datetime import datetime


FILE_NAME = "tasks.json"
VALID_STATUSES = {"todo", "in-progress", "done"}


def current_time():
    """Menghasilkan waktu saat ini."""
    return datetime.now().astimezone().isoformat(timespec="seconds")


def save_tasks(tasks):
    """Menyimpan seluruh task ke file JSON."""
    try:
        with open(FILE_NAME, "w", encoding="utf-8") as file:
            json.dump(tasks, file, indent=4, ensure_ascii=False)
    except OSError as error:
        print(f"Gagal menyimpan data: {error}")
        sys.exit(1)


def load_tasks():
    """Membaca task dari JSON dan membuat file jika belum tersedia."""
    if not os.path.exists(FILE_NAME):
        save_tasks([])
        return []

    try:
        with open(FILE_NAME, "r", encoding="utf-8") as file:
            tasks = json.load(file)

        if not isinstance(tasks, list):
            raise ValueError("Isi tasks.json harus berbentuk list.")

        return tasks

    except (OSError, json.JSONDecodeError, ValueError) as error:
        print(f"Gagal membaca tasks.json: {error}")
        sys.exit(1)


def find_task(tasks, task_id):
    """Mencari task berdasarkan ID."""
    for task in tasks:
        if task["id"] == task_id:
            return task

    return None


def add_task(tasks, description):
    """Menambahkan task baru."""
    description = description.strip()

    if not description:
        print("Deskripsi task tidak boleh kosong.")
        return

    new_id = max(
        (task["id"] for task in tasks),
        default=0
    ) + 1

    time_now = current_time()

    new_task = {
        "id": new_id,
        "description": description,
        "status": "todo",
        "createdAt": time_now,
        "updatedAt": time_now
    }

    tasks.append(new_task)
    save_tasks(tasks)

    print(f"Task added successfully (ID: {new_id})")


def update_task(tasks, task_id, new_description):
    """Mengubah deskripsi task."""
    task = find_task(tasks, task_id)

    if task is None:
        print(f"Task dengan ID {task_id} tidak ditemukan.")
        return

    new_description = new_description.strip()

    if not new_description:
        print("Deskripsi task tidak boleh kosong.")
        return

    task["description"] = new_description
    task["updatedAt"] = current_time()

    save_tasks(tasks)
    print(f"Task {task_id} berhasil diperbarui.")


def delete_task(tasks, task_id):
    """Menghapus task berdasarkan ID."""
    task = find_task(tasks, task_id)

    if task is None:
        print(f"Task dengan ID {task_id} tidak ditemukan.")
        return

    tasks.remove(task)
    save_tasks(tasks)

    print(f"Task {task_id} berhasil dihapus.")


def mark_task(tasks, task_id, new_status):
    """Mengubah status task."""
    task = find_task(tasks, task_id)

    if task is None:
        print(f"Task dengan ID {task_id} tidak ditemukan.")
        return

    task["status"] = new_status
    task["updatedAt"] = current_time()

    save_tasks(tasks)
    print(f"Status task {task_id} diubah menjadi {new_status}.")


def list_tasks(tasks, status=None):
    """Menampilkan seluruh task atau berdasarkan status."""
    if status == "not-done":
        selected_tasks = [
            task for task in tasks
            if task["status"] != "done"
        ]

    elif status is None:
        selected_tasks = tasks

    else:
        selected_tasks = [
            task for task in tasks
            if task["status"] == status
        ]

    if not selected_tasks:
        print("Tidak ada task yang sesuai.")
        return

    for task in selected_tasks:
        print("-" * 40)
        print(f'ID          : {task["id"]}')
        print(f'Description : {task["description"]}')
        print(f'Status      : {task["status"]}')
        print(f'Created     : {task["createdAt"]}')
        print(f'Updated     : {task["updatedAt"]}')


def convert_id(value):
    """Mengubah ID dari teks menjadi angka."""
    try:
        return int(value)
    except ValueError:
        print("ID harus berupa angka.")
        return None


def show_help():
    """Menampilkan panduan penggunaan."""
    print("""
PENGGUNAAN:

python task_cli.py add "deskripsi"
python task_cli.py update ID "deskripsi baru"
python task_cli.py delete ID
python task_cli.py mark-in-progress ID
python task_cli.py mark-done ID
python task_cli.py list
python task_cli.py list todo
python task_cli.py list in-progress
python task_cli.py list done
python task_cli.py list not-done
""")


def main():
    args = sys.argv[1:]

    if not args:
        show_help()
        return

    tasks = load_tasks()
    command = args[0]

    if command == "add" and len(args) == 2:
        add_task(tasks, args[1])

    elif command == "update" and len(args) == 3:
        task_id = convert_id(args[1])

        if task_id is not None:
            update_task(tasks, task_id, args[2])

    elif command == "delete" and len(args) == 2:
        task_id = convert_id(args[1])

        if task_id is not None:
            delete_task(tasks, task_id)

    elif command == "mark-in-progress" and len(args) == 2:
        task_id = convert_id(args[1])

        if task_id is not None:
            mark_task(tasks, task_id, "in-progress")

    elif command == "mark-done" and len(args) == 2:
        task_id = convert_id(args[1])

        if task_id is not None:
            mark_task(tasks, task_id, "done")

    elif command == "list" and len(args) in {1, 2}:
        status = args[1] if len(args) == 2 else None

        allowed_statuses = VALID_STATUSES | {"not-done"}

        if status is not None and status not in allowed_statuses:
            print("Status tidak valid.")
            print("Gunakan: todo, in-progress, done, atau not-done.")
            return

        list_tasks(tasks, status)

    else:
        print("Perintah atau jumlah argumen tidak valid.")
        show_help()


if __name__ == "__main__":
    main()
