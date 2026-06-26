# Mejoras al prompt de generación — Visual Styling

Evaluación realizada el 26/06/2026 comparando dos generaciones del mismo componente
(pricing card con tres tiers) antes y después del primer ajuste al prompt.

---

## Lo que mejoró con el primer ajuste

- **Fondo claro por defecto** — se eliminó el gradiente slate-900→slate-800. El canvas ahora es blanco, más limpio y versátil.
- **Tipografía con mayor contraste de tamaño** — el heading principal usa un tamaño más dramático y peso bold que se siente más intencional.
- **Badge menos agresivo** — pasó de amarillo (`bg-yellow-400`) a azul, y el texto de "POPULAR" a "RECOMMENDED".

---

## Lo que sigue igual y necesita atención

### 1. El patrón de lista con checkmarks persiste
El modelo simplemente cambió los checkmarks de verde a azul. La instrucción en el prompt prohíbe "green checkmarks" pero no el patrón en sí.

**Sugerencia:** Prohibir el patrón de lista con icono checkmark como mecanismo por defecto para features. Proponer alternativas: texto plano con separadores, puntos neutros, o ítems numerados.

### 2. El badge de esquina sigue existiendo
"RECOMMENDED" en azul sigue siendo el mismo cliché que "POPULAR" en amarillo; solo cambió el color y la palabra. El problema es la posición absoluta en la esquina superior derecha, no el texto.

**Sugerencia:** Prohibir explícitamente el patrón de badge posicionado absolutamente en la esquina de una card. Si se necesita destacar un tier, hacerlo con tratamiento estructural (escala, borde izquierdo de acento, tipografía en el nombre del tier).

### 3. Las tres cards son estructuralmente idénticas
Los tres tiers comparten exactamente el mismo template: nombre → descripción → precio → botón → lista de features. No hay ninguna diferencia estructural, solo de datos. El resultado se ve como una tabla disfrazada de cards.

**Sugerencia:** Indicar en el prompt que, cuando hay múltiples tiers, el tier destacado debería tener una composición visual distinta (no solo estilos diferentes sobre la misma estructura).

### 4. El azul por defecto de Tailwind sigue siendo el único acento
El modelo continúa usando `blue-600` / `blue-500` para botones, checkmarks, bordes y el badge. El primer ajuste prohibió el gradiente de azul para el fondo de la card destacada, pero el azul sigue dominando como único color de acento porque es el default de Tailwind.

**Sugerencia:** Añadir una instrucción explícita para que el modelo elija un color de acento que no sea el azul por defecto de Tailwind (`blue-*`), a menos que el usuario lo pida específicamente. Proponer que se explore indigo oscuro, slate neutro con un acento cálido (amber, rose), verde-esmeralda, etc.

### 5. Layout sigue siendo una sola columna apilada
Las cards se generan en columna vertical (mobile-first por defecto), lo cual en desktop se ve como una lista larga sin jerarquía visual clara. El prompt indica preferir layouts editoriales sobre card grids "cuando el contenido lo permita", pero un pricing table con 3 tiers es precisamente un caso donde la comparación horizontal tiene sentido.

**Sugerencia:** Ajustar la instrucción para no generalizar "evitar el grid de cards" — el problema del primer componente era que las tres cards eran visualmente planas e idénticas, no que estuvieran en un grid. El layout horizontal con un tier visualmente dominante es una solución válida y preferible a una columna apilada.

---

## Integración con GitHub Actions (pendiente de implementar)

Claude Code ofrece una integración oficial con GitHub que convierte al asistente en un miembro automatizado del equipo, capaz de responder menciones en issues/PRs y revisar pull requests automáticamente.

### Setup

1. Ejecutar `/install-github-app` en Claude Code (requiere API key configurada).
2. El comando instala la GitHub App y genera un PR con los archivos de workflow en `.github/workflows/`.
3. Al mergear ese PR, quedan activos los dos workflows por defecto.

> **Nota:** El intento del 26/06/2026 falló con `API key is required`. Pendiente de reintentar con la key configurada.

### Workflows disponibles

**Mention workflow** — Se activa cuando alguien menciona `@claude` en un issue o PR. Claude analiza la solicitud, planifica tareas, ejecuta con acceso al codebase y responde en el hilo.

**Pull Request review workflow** — Se activa automáticamente al abrir un PR. Claude revisa los cambios, analiza el impacto y publica un reporte detallado.

### Configuración recomendada para este proyecto

Una vez generado el PR inicial, personalizar los workflows con lo siguiente:

**Setup previo al run de Claude:**
```yaml
- name: Project Setup
  run: |
    npm run setup
    npm run dev:daemon
```

**Instrucciones contextuales para Claude:**
```yaml
custom_instructions: |
  The project is already set up with all dependencies installed.
  The server is already running at localhost:3000. Logs from it
  are being written to logs.txt. If needed, you can query the
  db with the 'sqlite3' cli. If needed, use the mcp__playwright
  set of tools to launch a browser and interact with the app.
```

**Configuración del MCP server de Playwright:**
```yaml
mcp_config: |
  {
    "mcpServers": {
      "playwright": {
        "command": "npx",
        "args": [
          "@playwright/mcp@latest",
          "--allowed-origins",
          "localhost:3000;cdn.tailwindcss.com;esm.sh"
        ]
      }
    }
  }
```

**Permisos de herramientas** — A diferencia del entorno local, en GitHub Actions cada herramienta debe listarse explícitamente en `allowed_tools`. No hay shortcuts. Ejemplo:
```yaml
allowed_tools: "Bash(npm:*),Bash(sqlite3:*),mcp__playwright__browser_snapshot,mcp__playwright__browser_click,..."
```

### Próximos pasos

- [ ] Configurar `ANTHROPIC_API_KEY` como secret en el repositorio de GitHub
- [ ] Ejecutar `/install-github-app` nuevamente
- [ ] Mergear el PR generado
- [ ] Personalizar los workflows con la configuración de arriba
- [ ] Probar con un issue simple mencionando `@claude`

---

### 6. El modelo sigue resumiendo lo que hizo en el mensaje final
El chat muestra: *"Done. Created a pricing card component with three tiers... The Pro tier is highlighted with a ring border and slight scale effect..."*. El prompt dice explícitamente que no resuma el trabajo a menos que el usuario lo pida.

Esto no es un problema de estilo visual, pero indica que esa instrucción no está teniendo el efecto esperado. Podría reforzarse con un ejemplo negativo en el prompt.

---

## Prioridad sugerida para el siguiente ajuste

| # | Mejora | Impacto visual |
|---|--------|----------------|
| 1 | Prohibir corner badge como patrón (no solo el color amarillo) | Alto |
| 2 | Prohibir checkmarks como lista de features por defecto | Alto |
| 3 | Forzar un acento que no sea `blue-*` por defecto | Alto |
| 4 | Clarificar que "evitar card grids" aplica a grids planos, no a layouts comparativos | Medio |
| 5 | Exigir diferencia estructural en el tier destacado, no solo visual | Medio |
