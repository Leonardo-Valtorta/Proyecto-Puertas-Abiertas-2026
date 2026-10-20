# Creación de archivo 📄 script.js 

## Indicaciones:
* En la carpeta raíz, crea un archivo con el nombre: **script.js**.
* En el archivo: **index.html** debajo de la etiqueta \</main\> y antes de cerrar la etiqueta \<body\> coloca el código del script:
  
```
  </main>
  
  <script src="script.js"></script>
</body>
```
* Dentro del archivo **script.js**, coloca el siguiente código:
```
const CAPACIDAD_DIARIA = 200;
const CLAVE_RESERVAS = "puertasAbiertasReservas";
const formulario = document.getElementById("form-reserva");
const campoFecha = document.getElementById("fecha");
const campoPersonas = document.getElementById("personas");
const avisoFechas = document.getElementById("rango-fechas");
const contenedorReservaciones = document.getElementById("reservaciones");

function formatearFecha(fecha) {
	const anio = fecha.getFullYear();
	const mes = String(fecha.getMonth() + 1).padStart(2, "0");
	const dia = String(fecha.getDate()).padStart(2, "0");

	return `${anio}-${mes}-${dia}`;
}

function leerReservas() {
	try {
		const reservas = JSON.parse(localStorage.getItem(CLAVE_RESERVAS) || "[]");
		return Array.isArray(reservas)
			? reservas.filter((reserva) =>
				reserva &&
				typeof reserva.fecha === "string" &&
				/^\d{4}-\d{2}-\d{2}$/.test(reserva.fecha) &&
				Number.isInteger(reserva.personas) &&
				reserva.personas > 0
			)
			: [];
	} catch {
		return [];
	}
}

function crearResumenPorFecha(reservas) {
	const resumen = new Map();

	reservas.forEach((reserva) => {
		const datos = resumen.get(reserva.fecha) || { cantidad: 0, personas: 0 };
		datos.cantidad += 1;
		datos.personas += reserva.personas;
		resumen.set(reserva.fecha, datos);
	});

	return resumen;
}

function renderizarTabla(reservas) {
	const resumen = crearResumenPorFecha(reservas);
	const titulo = document.createElement("h2");
	titulo.textContent = "Disponibilidad por fecha";

	const rango = document.createElement("p");
	rango.className = "rango-reservaciones";
	rango.textContent = `La tabla muestra fechas desde hoy hasta ${formatoLargo.format(fechaMaxima)}. Las fechas anteriores a hoy no se muestran.`;

	const desplazamiento = document.createElement("div");
	desplazamiento.className = "tabla-reservaciones-scroll";

	const tabla = document.createElement("table");
	tabla.className = "tabla-reservaciones";
	const encabezado = document.createElement("thead");
	const filaEncabezado = document.createElement("tr");

	["Fecha", "Reservaciones", "Lugares reservados", "Lugares disponibles"].forEach((texto) => {
		const columna = document.createElement("th");
		columna.scope = "col";
		columna.textContent = texto;
		filaEncabezado.append(columna);
	});

	encabezado.append(filaEncabezado);
	const cuerpo = document.createElement("tbody");
	const fechaFila = new Date(hoy);

	while (fechaFila <= fechaMaxima) {
		const fechaClave = formatearFecha(fechaFila);
		const datos = resumen.get(fechaClave) || { cantidad: 0, personas: 0 };
		const fila = document.createElement("tr");
		const valores = [
			formatoLargo.format(fechaFila),
			datos.cantidad,
			datos.personas,
			Math.max(0, CAPACIDAD_DIARIA - datos.personas)
		];

		valores.forEach((valor) => {
			const celda = document.createElement("td");
			celda.textContent = valor;
			fila.append(celda);
		});

		cuerpo.append(fila);
		fechaFila.setDate(fechaFila.getDate() + 1);
	}

	tabla.append(encabezado, cuerpo);
	desplazamiento.append(tabla);
	contenedorReservaciones.replaceChildren(titulo, rango, desplazamiento);
}

const hoy = new Date();
hoy.setHours(0, 0, 0, 0);

const fechaMaxima = new Date(hoy);
const diaDelMes = hoy.getDate();
fechaMaxima.setDate(1);
fechaMaxima.setMonth(fechaMaxima.getMonth() + 3);
const ultimoDiaMesMaximo = new Date(
	fechaMaxima.getFullYear(),
	fechaMaxima.getMonth() + 1,
	0
).getDate();
fechaMaxima.setDate(Math.min(diaDelMes, ultimoDiaMesMaximo));

const formatoLargo = new Intl.DateTimeFormat("es-AR", {
	day: "2-digit",
	month: "long",
	year: "numeric"
});

campoFecha.min = formatearFecha(hoy);
campoFecha.max = formatearFecha(fechaMaxima);
avisoFechas.textContent = `Puedes elegir una fecha desde hoy hasta ${formatoLargo.format(fechaMaxima)} (máximo tres meses desde la fecha actual).`;

formulario.addEventListener("submit", (evento) => {
	evento.preventDefault();

	const datosFormulario = new FormData(formulario);
	const fecha = campoFecha.value;
	const personas = Number(campoPersonas.value);
	const reservas = leerReservas();
	const resumen = crearResumenPorFecha(reservas);
	const personasReservadas = resumen.get(fecha)?.personas || 0;

	if (personasReservadas + personas > CAPACIDAD_DIARIA) {
		alert("No hay suficientes lugares disponibles para esa fecha.");
		return;
	}

	reservas.push({
		apellido: datosFormulario.get("apellido").trim(),
		nombre: datosFormulario.get("nombre").trim(),
		telefono: datosFormulario.get("telefono").trim(),
		fecha,
		personas
	});

	try {
		localStorage.setItem(CLAVE_RESERVAS, JSON.stringify(reservas));
	} catch {
		alert("No se pudo guardar la reservación en este navegador.");
		return;
	}

	formulario.reset();
	alert("Reservación registrada.");
	renderizarTabla(reservas);
});

renderizarTabla(leerReservas());
```
* Guarda el archivo y prueba realizar una reserva en el navegador.
* Cierra el navegador y vuélvelo a abrir con Go-Live.
 
  > a) ¿qué ha pasado?¿La reservación sigue guardada? 

   Si es así... 

  > b) ¿Dónde se ha guardado?

	> c) Los datos, ¿deberían estar guardados en una base de datos?