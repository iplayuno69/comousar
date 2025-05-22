---

# 📄 Documentación de `api_server.js`

Este archivo implementa un servidor en Node.js que actúa como una **API REST** para:

1. Generar una **frase mnemónica** (seed) usando el estándar BIP39.
2. Derivar una **cartera de Bitcoin Cash (BCH)** con claves y direcciones a partir de esa frase.

---

## 📦 Dependencias

```js
const express = require('express');          // Framework web
const bodyParser = require('body-parser');   // Parseo de JSON
const cors = require('cors');                // Habilita CORS
const path = require('path');                // Manejo de rutas (no usado)

const bip39 = require('bip39');              // Generación y validación de frases mnemónicas
const { payments } = require('bitcoinjs-lib'); // Generación de direcciones Bitcoin
const { ECPairFactory } = require('ecpair'); // Generación de claves
const bchaddr = require('bchaddrjs');        // Conversión a direcciones BCH (CashAddr)
const HDKey = require('hdkey');              // Derivación HD desde la semilla

const ecc = require('tiny-secp256k1');       // Criptografía ECC (secp256k1)
const ECPair = ECPairFactory(ecc);           // Par de claves ECC
```

---

## 🔧 Función: `generateBCHWallet(mnemonicPhrase, index = 0)`

Esta función recibe:

* `mnemonicPhrase`: una frase mnemónica BIP39 de 12 palabras.
* `index`: el índice de la derivación HD (por defecto 0).

### ✔️ ¿Qué hace?

1. Valida la frase mnemónica.
2. Convierte la frase en una **semilla binaria**.
3. Deriva una clave según el estándar **BIP44 para BCH**: `m/44'/145'/0'/0/index`.
4. Genera:

   * Clave privada (hex)
   * Clave pública (hex)
   * Dirección `legacy` (formato antiguo)
   * Dirección `CashAddr` (moderno)

### 📤 Retorna:

```json
{
  "mnemonic": "la frase mnemónica usada",
  "path": "ruta BIP44",
  "privateKey": "clave privada en hex",
  "publicKey": "clave pública en hex",
  "legacyAddress": "dirección en formato Bitcoin clásico",
  "cashAddress": "dirección en formato Bitcoin Cash (CashAddr)"
}
```

---

## 🌐 Endpoints de la API

### `POST /generate-bch-wallet`

**Entrada (JSON):**

```json
{
  "mnemonic": "frase mnemónica BIP39",
  "index": 0
}
```

**Salida (JSON):** Objeto con claves y direcciones (ver más arriba).

**Errores comunes:**

* 400: Falta la frase mnemónica.
* 500: La frase no es válida o ocurrió un error.

---

### `GET /generate-mnemonic`

**Función:** Genera una nueva frase mnemónica aleatoria de 12 palabras.

**Salida:**

```json
{
  "mnemonic": "doce palabras en inglés"
}
```

---

## 🚀 Inicio del servidor

```js
app.listen(port, () => {
    console.log(`Servidor de API BCH ejecutándose en http://localhost:${port}`);
    console.log('Usando la wordlist estándar de BIP39 (inglés) para todas las operaciones.');
});
```

El servidor se ejecuta en el **puerto 3000** (puede ser remapeado por tu panel como Plesk, cPanel, etc.).

---



index.php:


---

## ✅ ¿Qué hace este archivo?

* Permite **generar una frase mnemónica BIP39 aleatoria** (vía `fetch` JS → Node.js).
* Permite **enviar esa frase + un índice** al servidor PHP.
* El PHP envía esos datos al servidor Node.js, que devuelve claves y direcciones BCH derivadas.

---

## 🔍 Secciones clave del archivo:

### 1. HTML (Formulario)

```html
<form method="post" onsubmit="return validateForm()">
    ...
    <textarea id="mnemonic" name="mnemonic" ...></textarea>
    ...
    <input type="number" id="initial_index" name="initial_index" ...>
    <input type="number" id="num_wallets" name="num_wallets" ...>
    ...
    <button type="button" onclick="generateRandomMnemonic()">Generar Frase Aleatoria</button>
    <input type="submit" name="generate" value="Generar Wallets BCH">
</form>
```

* Si haces clic en **"Generar Frase Aleatoria"**, se ejecuta JS (`generateRandomMnemonic`).
* Si haces clic en **"Generar Wallets BCH"**, se envía el formulario al servidor PHP para procesar.

---

### 2. JavaScript (cliente)

#### a) `generateRandomMnemonic()`

```javascript
const url = 'http://localhost:3000/generate-mnemonic';
const response = await fetch(url);
const data = await response.json();
document.getElementById('mnemonic').value = data.mnemonic;
```

* Llama al servidor Node.js para obtener una frase aleatoria.
* Llena el `textarea` con esa frase.

#### b) `validateForm()`

```javascript
if (mnemonicField.value.trim() === '') {
    alert('Por favor, ingresa o genera una frase mnemónica antes de continuar.');
    return false;
}
```

* Evita que se envíe el formulario si no hay frase mnemónica.

---

### 3. PHP (procesamiento)

```php
if (isset($_POST['generate'])) {
```

* Solo se ejecuta si se envió el formulario (por POST) y se presionó el botón.

#### Validaciones básicas:

```php
if (empty($mnemonic)) ...
elseif ($numWallets <= 0) ...
```

* Verifica que se haya ingresado una frase.
* Verifica que el número de wallets sea válido.

---

### 4. Ciclo de generación de direcciones

```php
for ($i = 0; $i < $numWallets; $i++) {
    $currentIndex = $initialIndex + $i;
    ...
    $result = file_get_contents($apiUrl, false, $context);
```

* Por cada dirección que se desea generar:

  * Se prepara la solicitud con `mnemonic` e `index`.
  * Se llama a la API Node.js.
  * Se imprime el resultado (clave privada, pública, dirección).

---

### 5. Manejo de errores

```php
if ($result === FALSE) {
    ...
} elseif (isset($walletInfo['error'])) {
    ...
} else {
    // Muestra claves y direcciones.
}
```

* Si el servidor no responde o devuelve error, muestra advertencias.

---

## ✅ Qué debes tener funcionando

1. **El servidor Node.js** debe estar corriendo en `http://localhost:3000`.
2. **Debe tener los endpoints**:

   * `GET /generate-mnemonic`
   * `POST /generate-bch-wallet`
3. Este archivo PHP puede estar en un servidor local (ej. `localhost:8000`) o Apache.

---
