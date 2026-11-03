# ¿Cuánto gasta la gente en Uruguay?

Página para que cualquier persona suba su estado de cuenta, vea en qué gasta y lo compare con otras personas parecidas, sin crear cuenta y sin compartir datos personales.

- **El estado de cuenta nunca sale del dispositivo de la persona.** La página lo lee en el navegador.
- **Solo se envía un resumen anónimo**: mes, rango de edad, tipo de hogar, departamento, rango de ingreso (opcional) y montos por categoría redondeados a $ 100.
- **Los resultados muestran solo medianas de grupos de 3 aportes o más.**

Archivos:

| Archivo | Qué es |
|---|---|
| `index.html` | La página completa. Es lo que subís a GitHub. |
| `supabase.sql` | La configuración de la base de datos. Se pega una vez en Supabase. |
| `README.md` | Esta guía. |

---

## Paso 1: crear la base de datos en Supabase (10 minutos)

1. Entrá a **supabase.com** y creá una cuenta. Podés usar tu cuenta de GitHub o de Google.
2. Tocá **New project**.
   - **Name:** `gastos-uy`
   - **Database password:** inventá una contraseña larga y guardala. No la vas a usar en la página.
   - **Region:** la más cercana. *South America (São Paulo)* es una buena opción.
   - Elegí el plan gratuito y tocá **Create new project**. Tarda un par de minutos.
3. En el menú de la izquierda, entrá a **SQL Editor** y tocá **New query**.
4. Abrí el archivo `supabase.sql`, copiá todo el contenido, pegalo y tocá **Run**. Tiene que aparecer *Success. No rows returned*.
   - Si aparece un error, copiá el mensaje completo y pedime ayuda.
5. Andá a **Project Settings → API Keys** (o **Project Settings → API**, según la versión) y copiá dos cosas:
   - la **Project URL**, que se parece a `https://abcdefgh.supabase.co`;
   - la clave **publishable** (empieza con `sb_publishable_`) o, si no aparece, la clave **anon public** (empieza con `eyJ`).

> ⚠️ **Nunca** copies a la página la clave **secret** ni la **service_role**. Esas dan acceso total a la base de datos. La clave publishable o anon está pensada para ser pública: la seguridad la dan las reglas de `supabase.sql`.

## Paso 2: conectar la página

Abrí `index.html` con cualquier editor de texto (el Bloc de notas sirve) y buscá estas líneas, casi al principio del código:

```js
const SUPABASE_URL = "";
const SUPABASE_KEY = "";
const RESPONSABLE  = "";
```

Completalas así, con tus datos:

```js
const SUPABASE_URL = "https://abcdefgh.supabase.co";
const SUPABASE_KEY = "sb_publishable_xxxxxxxxxxxx";
const RESPONSABLE  = "Tu nombre — tumail@ejemplo.com";
```

Guardá el archivo.

## Paso 3: publicarla en GitHub Pages (5 minutos)

1. Entrá a **github.com** y creá una cuenta si no tenés.
2. Arriba a la derecha tocá **+ → New repository**.
   - **Repository name:** `gastos-uy`
   - Marcá **Public**. GitHub Pages es gratis para repositorios públicos.
   - Tocá **Create repository**.
3. En el repositorio nuevo, tocá **uploading an existing file** (o **Add file → Upload files**), arrastrá `index.html` y `README.md`, y tocá **Commit changes**.
   - **No subas** el estado de cuenta de nadie ni archivos con datos reales.
4. Andá a **Settings → Pages**.
   - En **Source** elegí **Deploy from a branch**.
   - En **Branch** elegí `main` y la carpeta `/ (root)`, y tocá **Save**.
5. Esperá uno o dos minutos y recargá. Arriba aparece la dirección de tu página, algo como `https://tu-usuario.github.io/gastos-uy/`.

Ese es el link que compartís.

## Paso 4: probarla

1. Abrí el link, subí tu estado de cuenta y revisá el paso 2.
2. Completá tus datos en el paso 3, marcá la casilla de consentimiento y tocá **Enviar aporte**.
3. En Supabase, entrá a **Table Editor → aportes**. Tiene que aparecer una fila con tu resumen.
4. Los resultados aparecen cuando hay al menos 3 aportes del mismo mes.

## Cómo cambiar algo después

Editá `index.html` en tu computadora y volvé a subirlo a GitHub con **Add file → Upload files**. Se reemplaza solo. También podés tocar el archivo en GitHub y después el lápiz para editarlo ahí mismo.

## Moderar aportes

En Supabase, **Table Editor → aportes**, podés ver todas las filas y borrar las que parezcan falsas (seleccionás la fila → **Delete**). No vas a ver quién envió cada una: solo hay una huella del código al azar de su dispositivo.

Algunas consultas útiles para pegar en el **SQL Editor**:

```sql
-- cuántos aportes hay por mes
select mes, count(*) from aportes group by mes order by mes desc;

-- aportes con totales sospechosos
select * from aportes where total < 2000 or total > 500000 order by total;

-- borrar un aporte por su id
delete from aportes where id = 123;
```

## Cómo se protege la privacidad

- La página nunca envía el archivo. Se puede comprobar en el código, que es público.
- Antes de enviar, la persona ve su documento con nombre, dirección y números de cuenta tapados, y el resumen exacto que se envía.
- En la base de datos no hay nombres, mails, comercios, fechas exactas ni números de cuenta.
- La página no puede leer la tabla directamente: solo puede llamar a cuatro funciones (enviar, ver el propio, borrar el propio y ver resultados agrupados).
- Cada aporte queda ligado a un código al azar guardado en el navegador de la persona. En la base solo se guarda una huella (hash) de ese código. Si la persona borra los datos del navegador, ya no puede editar su aporte, pero tampoco nadie puede vincularlo con ella.

## Limitaciones a tener en cuenta

- **Aportes falsos.** Como no hay cuentas, alguien podría enviar datos inventados o muchos aportes. La base de datos limita los envíos por minuto, pero conviene revisar los datos de vez en cuando. Si crece mucho, se puede agregar un control anti-robots como Cloudflare Turnstile.
- **Supabase gratis.** En el plan gratuito, los proyectos pueden pausarse si pasan un tiempo sin actividad. Si la página deja de mostrar resultados, entrá a Supabase y reactivá el proyecto.
- **Grupos chicos.** Con pocos aportes, los filtros muy específicos no muestran nada. Es intencional, para que nadie pueda deducir el gasto de otra persona.
- **Lectura de estados de cuenta.** Está probada con un estado de cuenta de tarjeta Visa de Itaú y con movimientos de cuenta de Itaú Link. Otros bancos pueden necesitar ajustes. Si un archivo no se lee, la persona puede completar los montos a mano.

## Tema legal

La Ley 18.331 de protección de datos personales pide consentimiento informado y la inscripción de las bases de datos con datos personales ante la URCDP (datospersonales.gub.uy). Esta página guarda solo resúmenes anónimos y pide consentimiento explícito, pero no es asesoramiento legal: si pensás juntar muchos aportes o usarlos para algo más que estadísticas, consultá con la URCDP o con un abogado.
