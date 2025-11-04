# Doctrine: Schema Tool vs Migrations

## 1. Comandaments

### Paquet “Schema Tool” (canvia l’esquema directament)

```bash
# Comprova que el mapping (entitats) concorda amb la BD
php bin/console doctrine:schema:validate

# Mostra el SQL necessari per sincronitzar BD ←→ entitats (no executa)
php bin/console doctrine:schema:update --dump-sql

# EXECUTA els canvis directament a la BD (sense historial)
php bin/console doctrine:schema:update --force
```

- **validate** → només verifica inconsistències (ideal per a CI).
- **update --dump-sql** → mostra què passaria (segur per a revisar abans d’executar).
- **update --force** → aplica els canvis *in situ*, però **no deixa rastre** ni permet rollback. Pot provocar pèrdua de dades si hi ha `DROP` o `ALTER`.

---

### Paquet “Migrations” (historial versionat d’esquema)

```bash
# Genera un fitxer de migració amb els canvis detectats
php bin/console make:migration

# Aplica (i registra) les migracions pendents
php bin/console doctrine:migrations:migrate
```

- **make:migration** → crea un fitxer PHP amb el SQL necessari (*diff controlat*). Pots revisar, editar i versionar-lo al repositori.
- **migrate** → executa de forma segura i registra quina versió d’esquema tens en cada entorn, permetent rollback.

> 💡 *`make:migration` (de MakerBundle) usa internament el motor de Doctrine Migrations. És el flux recomanat.*

---

## 2. Quines són millors?

### Per a projectes reals (equip, integració contínua, pre/producció):
👉 **MIGRACIONS (`make:migration` + `migrate`)** són l’**estàndard**.

- Manté **traça d’històric**.
- Permet **revisions (PR)** i **rollback**.
- Assegura **consistència entre entorns**.

### Per a prototips o proves locals ràpides:
👉 **`schema:update --dump-sql`** per a veure els canvis.  
Usa **`--force` només en local** i si acceptes el risc (⚠️ **mai en producció**).

---

## 3. Quan utilitzar cadascuna

| Situació | Comandes recomanades | Per què |
|-----------|----------------------|----------|
| **Desenvolupament d’equip amb Git/CI** | `make:migration` → revisió PR → `migrate` | Historial versionat i reproduïble. |
| **Desplegar a staging/producció** | `doctrine:migrations:migrate` (automatitzat) | Segur i traçable. Evita divergències. |
| **Verificar coherència en CI** | `doctrine:schema:validate` | Falla ràpid si mapping ≠ BD. |
| **Spike / prova local ràpida** | `schema:update --dump-sql` (i només si cal `--force`) | Agilitat, però sense historial. |

---

## 4. Resum

- ✅ Sempre que hi haja repo i més d’un entorn → MIGRACIONS.
- ⚠️ Evita `schema:update --force` fora de local/proves.
- 🔍 Usa `schema:validate` en CI per detectar problemes de mapping.

