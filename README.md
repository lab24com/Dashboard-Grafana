# Dashboards de Grafana para agentes Zabbix | LAKA

**Desarrollado por Willian Tola — LAKA Soluciones Tecnológicas**

Primera versión pública de dashboards para visualizar servidores Windows y Linux monitoreados mediante agentes Zabbix. Incluye una vista principal y una vista de detalle por servidor, con una interfaz oscura pensada para el trabajo del NOC.

Este repositorio contiene dashboards JSON; los plugins se instalan por separado desde el catálogo de Grafana.

## Funcionalidades

- CPU usada, número de CPU y memoria RAM usada y total.
- Uso de discos y tráfico de red recibido y enviado.
- Sistema operativo, versión del agente y tiempo activo.
- Problemas activos de Zabbix y filtros por estado y sistema operativo.
- Búsqueda de servidores y ordenamiento de columnas en el principal.
- Acceso al detalle al hacer clic en el nombre del servidor y botón de regreso.
- Gauges para CPU y RAM, gráficas históricas y representación circular de discos en el detalle.

## Archivos

| Archivo | Función |
| --- | --- |
| [LAKA_Dashboard_Agentes_Zabbix_Principal.json](LAKA_Dashboard_Agentes_Zabbix_Principal.json) | Resumen de los servidores y filtros. |
| [LAKA_Dashboard_Agentes_Zabbix_Detalle.json](LAKA_Dashboard_Agentes_Zabbix_Detalle.json) | Métricas y gráficas del servidor seleccionado. |

## Requisitos y plugins

Configuración de referencia para estos archivos:

| Componente | Requisito |
| --- | --- |
| Grafana | 11 o 12, compatibles con Business Text 6.x. |
| Zabbix | Plantillas de agentes Windows y Linux de las ramas 7.0 / 7.4; revisar nombres y claves si se usan versiones diferentes. |
| Plugin Zabbix | `alexanderzobnin-zabbix-app`, instalado y habilitado; usar una versión compatible con Grafana y Zabbix. Los JSON declaran la versión 5.0.0 como referencia. |
| Plugin Business Text | `marcusolsson-dynamictext-panel`, versión 6.x. Se utiliza en ambos dashboards. |
| Fuente de datos | Zabbix configurado en Grafana, con acceso a los hosts y a la API. |

Los paneles Gauge, Stat y Time series están incluidos en Grafana. No se necesita un plugin adicional para los discos: su representación circular está implementada en Business Text.

Las versiones declaradas en los JSON no representan una certificación de todas las combinaciones de versiones. Confirme los requisitos del plugin antes de instalarlo o actualizarlo.

### Plantillas admitidas por el principal

El dashboard principal verifica que el host tenga vinculada directamente al menos una de estas plantillas:

- `Windows by Zabbix agent`
- `Windows by Zabbix agent active`
- `Linux by Zabbix agent`
- `Linux by Zabbix agent active`

Los equipos monitoreados únicamente mediante SNMP o HTTP no forman parte de esta vista. Si usa plantillas renombradas, personalizadas o heredadas mediante otra plantilla, tendrá que adaptar la validación del principal.

## 1. Instalar los plugins

### Desde la interfaz de Grafana

1. Ingrese con una cuenta administradora.
2. Abra **Administration → Plugins and data → Plugins**; el nombre del menú puede variar según la versión.
3. Busque e instale **Zabbix**. Luego abra el plugin y pulse **Enable**.
4. Busque e instale **Business Text**, en una versión 6.x compatible.

### Desde Linux

Para una instalación de Grafana administrada como servicio, puede usar estos comandos. Instalan las versiones disponibles en el catálogo; confirme su compatibilidad antes de ejecutarlos:

```bash
sudo grafana cli plugins install alexanderzobnin-zabbix-app
sudo grafana cli plugins install marcusolsson-dynamictext-panel
sudo systemctl restart grafana-server
```

Si su instalación usa el ejecutable antiguo, sustituya `grafana cli` por `grafana-cli`. Para fijar una versión, agregue su número al final del comando de instalación.

En Docker o Grafana Cloud, instale los plugins mediante el mecanismo correspondiente a su despliegue. No ejecute `systemctl` dentro de un contenedor.

## 2. Configurar la fuente de datos Zabbix

1. Abra **Connections → Data sources → Add data source**.
2. Seleccione **Zabbix**.
3. Configure la URL completa de la API, por ejemplo:

   ```text
   https://zabbix.example.com/zabbix/api_jsonrpc.php
   ```

   Si su frontend no está publicado bajo `/zabbix`, ajuste esa ruta.

4. Configure usuario y contraseña, o un token de API si su versión del plugin lo admite.
5. Use una cuenta con permisos de lectura sobre los grupos y hosts que desea mostrar. Su rol también debe permitir las consultas API utilizadas: `host.get`, `item.get` y `history.get`.
6. Pulse **Save & test** y confirme que la conexión funciona.

Las credenciales se configuran en la fuente de datos de Grafana; no deben agregarse al código del dashboard. La conexión directa a la base de datos de Zabbix no es obligatoria.

## 3. Importar los dashboards

1. Descargue ambos archivos JSON del repositorio.
2. Abra **Dashboards → New → Import**.
3. Cargue primero `LAKA_Dashboard_Agentes_Zabbix_Detalle.json`.
4. En el campo **Zabbix**, seleccione la fuente de datos creada anteriormente y pulse **Import**.
5. Repita el proceso con `LAKA_Dashboard_Agentes_Zabbix_Principal.json`, seleccionando la misma fuente de datos.
6. Abra el principal y seleccione **Últimas 2 horas**.
7. Haga clic en un servidor para abrir su detalle.

Mantenga los UID originales para conservar los enlaces entre ambos dashboards:

| Dashboard | UID |
| --- | --- |
| Principal | `zbx-agents-resources-v3` |
| Detalle | `zbx-win-agent-detail-plugin-v2` |

Aunque el UID del detalle conserva la palabra `win` por compatibilidad, la vista está destinada a Windows y Linux.

Si ya existe un dashboard con el mismo UID, Grafana puede solicitar reemplazarlo. Para actualizar esta misma instalación, seleccione la fuente de datos y reemplace el dashboard correspondiente.

El HTML, CSS y JavaScript de los paneles Business Text están incluidos en los JSON. No necesita copiarlos manualmente ni instalar un módulo en el frontend de Zabbix.

## 4. Validar la instalación

- Los hosts deben tener ítems habilitados y datos recientes en **Monitoring → Latest data** de Zabbix.
- Compruebe CPU, RAM, discos, tráfico, sistema operativo, agente y uptime de un host Windows y otro Linux.
- Pruebe búsqueda, filtros, acceso al detalle y regreso al principal.
- Para comparar valores entre Grafana y Zabbix, use el mismo horario.

## Interpretación de los datos

- **CPU y RAM:** porcentaje usado. **RAM total:** GiB.
- **Discos:** porcentaje usado; en el detalle también se representa el porcentaje libre.
- **Red:** bits por segundo, con escala automática en el principal. La suma puede incluir interfaces virtuales.
- **Uptime:** tiempo activo del sistema.
- **Umbrales del principal:** advertencia desde 75 % y crítico desde 90 % para CPU, RAM y discos. Los problemas de Zabbix también influyen en el estado.
- **`—`:** sin lectura válida o reciente para esa métrica. No equivale automáticamente a un servidor caído.

El principal considera recientes las métricas dinámicas de los últimos 15 minutos respecto al final del período seleccionado. Datos como versión del agente y sistema operativo se consultan como metadatos actuales. Una vista histórica puede, por tanto, mostrar métricas del período junto a metadatos actuales.

La etiqueta **Normal** no garantiza que todas las columnas tengan datos: revise también las celdas sin lectura. El mantenimiento solo se identifica cuando viene informado en los datos de problemas.

## Solución de problemas

| Problema | Qué revisar |
| --- | --- |
| Panel vacío o plugin no encontrado | Instalación de Business Text 6.x y Zabbix, habilitación de Zabbix y reinicio cuando corresponda. |
| No aparece un servidor | Permisos de lectura, selección de hosts y vinculación directa con una de las cuatro plantillas admitidas. |
| Faltan RAM o uptime | Últimos datos, claves de los ítems, estado de los ítems, frecuencia de actualización y rango horario. Las plantillas personalizadas pueden requerir adaptar consultas. |
| Error al verificar las plantillas | Conexión y permisos de API mediante la fuente de datos Zabbix. |
| No funcionan los enlaces | Importación de ambos dashboards y conservación de sus UID. |
| Actualización lenta | Comience con Últimas 2 horas y actualización cada minuto; revise cantidad de hosts, interfaces y tiempos de consulta. |
| Zabbix muestra un valor diferente | Compare unidades, hora de la muestra y el mismo intervalo. Un rango absoluto en Grafana no avanza al actualizar. |

Si el problema continúa, abra un **Issue** con versiones de Grafana, Zabbix y plugins, plantilla utilizada, descripción del fallo y capturas sin información sensible. Para una métrica ausente, incluya su nombre, clave y hora de última lectura.

## Desarrollo y aportes

Esta es la primera versión pública y seguirá mejorando con las pruebas y comentarios de la comunidad. Las consultas y la presentación pueden requerir ajustes para cada entorno.

**Autor:** Willian Tola
**Marca:** LAKA Soluciones Tecnológicas
**Contact:** +591 70806592

Los plugins Grafana Zabbix y Business Text pertenecen a sus respectivos proyectos. LAKA desarrolla la configuración, la lógica y la presentación de estos dashboards.

## Documentación oficial

- [Plugin Zabbix para Grafana](https://grafana.com/grafana/plugins/alexanderzobnin-zabbix-app/)
- [Configuración de la fuente de datos Zabbix](https://grafana.com/docs/plugins/alexanderzobnin-zabbix-app/latest/configure/)
- [Business Text: requisitos e instalación](https://grafana.com/docs/plugins/marcusolsson-dynamictext-panel/latest/)
