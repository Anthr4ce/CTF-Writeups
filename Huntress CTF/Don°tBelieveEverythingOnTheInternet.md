```text
Don't believe everything you see on the Internet!  
  
Anyway, have you heard this intro soundtrack from Half-Life 3?
```


Le challenge commence par le téléchargement d'un fichier .mp3. En essayant de le traiter en Python pour extraire les métadonnées, j'ai rencontré des erreurs. Après plusieurs tentatives, je me suis demandé s'il ne s'agissait pas d'un vrai MP3, mais d'autre chose. J'ai donc écrit une fonction pour déterminer le vrai type du fichier:

```python
```python
 def check_file_type(filename):  
     kind = filetype.guess(filename)  
    if kind is None:  
        return "type de fichier inconnu."  
    return f"type: {kind.mime}, extension: {kind.extension}"  
  
 
 filename = 'src/Half-Life_3_OST.mp3'  
 print(check_file_type(filename))
```

Grâce à ce script, j'ai pu constater qu'en effet, le fichier était un PNG:
```bash
Type: image/png, Extension: png
```

J'ai ensuite traité l'image avec la bibliothèque PIL en utilisant ce code:

```python
from PIL import Image  
  
image_path = 'src/Half-Life_3_OST.mp3'  
img = Image.open(image_path)  
img.show()
```

Cela m'a permis d'obtenir le flag:
```t
flag{a85466991f0a8dc3d9837a5c32fa0c91}
```