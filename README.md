const readline = require("readline");

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});
function Tarea(titulo, descripcion, vencimiento) {
  this.titulo = titulo;
  this.descripcion = descripcion;
  this.estado = "Pendiente";
  this.dificultad = "+--";
  this.vencimiento = vencimiento;
  this.creacion = new Date().toLocaleDateString("es-AR");
}
function GestorDeTareas(nombreUsuario) {
  this.nombre = nombreUsuario;
  this.tareas = [];
  this.MAX_TAREAS = 100;
  this.rl = rl;
}
GestorDeTareas.prototype.limpiarPantalla = function() {
  console.clear();
};

GestorDeTareas.prototype.preguntar = function(pregunta) {
  return new Promise((resolve) => this.rl.question(pregunta, resolve));
};

GestorDeTareas.prototype.pausar = async function() {
  await this.preguntar("\nPresiona ENTER para continuar...");
};
GestorDeTareas.prototype.mostrarMenuPrincipal = async function() {
  this.limpiarPantalla();
  console.log(`¡Hola ${this.nombre}!\n`);
  console.log("¿Qué deseas hacer?\n");
  console.log("[1] Ver Mis Tareas");
  console.log("[2] Buscar una Tarea");
  console.log("[3] Agregar una Tarea");
  console.log("[0] Salir\n");

  let opcion = await this.preguntar("> ");
  switch (opcion.trim()) {
    case "1":
      await this.mostrarmenuFiltrarTareas();
      break;
    case "2":
      await this.buscarTareaPorTitulo();
      break;
    case "3":
      await this.crearTarea();
      break;
    case "0":
      console.log("Cerrando menu...");
      this.rl.close();
      process.exit(0);
    default:
      console.log("Opción no valida.");
      await this.pausar();
      await this.mostrarMenuPrincipal();
  }
};

GestorDeTareas.prototype.mostrarmenuFiltrarTareas = async function() {
  this.limpiarPantalla();
  console.log("¿Qué tareas deseas ver?\n");
  console.log("[1] Todas");
  console.log("[2] Pendientes");
  console.log("[3] En curso");
  console.log("[4] Terminadas");
  console.log("[0] Volver\n");

  let opcion = await this.preguntar("> ");
  switch (opcion.trim()) {
    case "1":
      await this.listarTareas();
      break;
    case "2":
      await this.listarTareas("Pendiente");
      break;
    case "3":
      await this.listarTareas("En curso");
      break;
    case "4":
      await this.listarTareas("Terminada");
      break;
    case "0":
      await this.mostrarMenuPrincipal();
      return;
    default:
      console.log("Opción no válida.");
      await this.pausar();
  }
  await this.mostrarmenuFiltrarTareas();
};

GestorDeTareas.prototype.listarTareas = async function(filtro = null) {
  this.limpiarPantalla();
  console.log(`--- ${filtro ? "Tareas " + filtro : "Todas tus tareas"} ---\n`);
  
  let lista = this.tareas
    .map((t, i) => ({ ...t, index: i }))
    .filter((t) => (filtro ? t.estado === filtro : true));

  if (lista.length === 0) {
    console.log("No hay tareas para ver.");
    await this.pausar();
    return;
  }

  lista.forEach((t, i) => {
    console.log(`[${i + 1}] ${t.titulo} | Estado: ${t.estado} | Vencimiento: ${t.vencimiento}`);
  });

  let sel = await this.preguntar("\n¿Ver detalles de alguna tarea? (número o 0 para volver): ");
  let seleccion = parseInt(sel);
  if (seleccion > 0 && seleccion <= lista.length) {
    await this.mostrarDetalleTarea(lista[seleccion - 1].index);
  }
};
GestorDeTareas.prototype.buscarTareaPorTitulo = async function() {
  this.limpiarPantalla();
  let query = await this.preguntar("Introduce el título de la tarea:\n> ");
  let resultados = this.tareas
    .map((t, i) => ({ ...t, index: i }))
    .filter((t) => t.titulo.toLowerCase().includes(query.toLowerCase()));
  if (resultados.length === 0) {
    console.log("No se encontraron tareas.");
    await this.pausar();
    await this.mostrarMenuPrincipal();
    return;
  }
  console.log("- Resultados -");
  resultados.forEach((t, i) => console.log(`[${i + 1}] ${t.titulo}`));
  let sel = await this.preguntar("\n¿Ver detalles de alguna? (número o 0 para volver): ");
  let seleccion = parseInt(sel);
  if (seleccion > 0 && seleccion <= resultados.length) {
    await this.mostrarDetalleTarea(resultados[seleccion - 1].index);
  } else {
    await this.mostrarMenuPrincipal();
  }
  await this.mostrarMenuPrincipal();
};

GestorDeTareas.prototype.crearTarea = async function() {
  if (this.tareas.length >= this.MAX_TAREAS) {
    console.log("No se pueden agregar más tareas. Límite alcanzado.");
    await this.pausar();
    await this.mostrarMenuPrincipal();
    return;
  }

  this.limpiarPantalla();
  let titulo = await this.preguntar("Título de la tarea:\n> ");
  let descripcion = await this.preguntar("Descripción:\n> ");
  let vencimiento = await this.preguntar("Fecha de vencimiento (DD/MM/AAAA):\n> ");
  
  let nueva = new Tarea(titulo, descripcion, vencimiento);
  this.tareas.push(nueva);
  console.log(`\n¡Tarea '${titulo}' creada exitosamente!`);
  await this.pausar();
  await this.mostrarMenuPrincipal();
};

GestorDeTareas.prototype.mostrarDetalleTarea = async function(index) {
  let t = this.tareas[index];
  this.limpiarPantalla();
  console.log("--- Detalle de la Tarea ---\n");
  console.log(`Título:       ${t.titulo}`);
  console.log(`Descripción: ${t.descripcion}`);
  console.log(`Estado:       ${t.estado}`);
  console.log(`Dificultad:  ${t.dificultad}`);
  console.log(`Vencimiento: ${t.vencimiento}`);
  console.log(`Creación:    ${t.creacion}`);
  console.log("\n[E] Editar | [D] Eliminar | [0] Volver");
  let op = await this.preguntar("> ");

  if (op.toLowerCase() === "e") {
    await this.editarTarea(index);
    await this.mostrarDetalleTarea(index);
  } else if (op.toLowerCase() === "d") {
    let confirm = await this.preguntar("¿Quieres eliminarla? : ");
    if (confirm.toLowerCase() === "s") {
      this.tareas.splice(index, 1);
      console.log("Tarea eliminada.");
      await this.pausar();
    }
    await this.mostrarMenuPrincipal();
  }
};

GestorDeTareas.prototype.editarTarea = async function(index) {
  let t = this.tareas[index];

  let desc = await this.preguntar(`Nueva descripción (ENTER para mantener actual: ${t.descripcion}):> `);
  if (desc.trim() !== "") t.descripcion = desc;

  let est = await this.preguntar("Nuevo estado [P]endiente/[E]n curso/[T]erminada/[C]ancelada:\n> ");
  switch (est.toLowerCase()) {
    case "p":
      t.estado = "Pendiente";
      break;
    case "e":
      t.estado = "En curso";
      break;
    case "t":
      t.estado = "Terminada";
      break;
    case "c":
      t.estado = "Cancelada";
      break;
  }

  let dif = await this.preguntar("Nueva dificultad [1] fácil/[2] media/[3] difícil:\n> ");
  if (dif === "1") t.dificultad = "+--";
  else if (dif === "2") t.dificultad = "++-";
  else if (dif === "3") t.dificultad = "+++";

  let venc = await this.preguntar(`Nuevo vencimiento (actual ${t.vencimiento}):\n> `);
  if (venc.trim() !== "") t.vencimiento = venc;

  console.log("Tarea actualizada");
  await this.pausar();
};

GestorDeTareas.prototype.iniciar = async function() {
  await this.mostrarMenuPrincipal();
};
const gestor = new GestorDeTareas("Olivia");
gestor.iniciar();
