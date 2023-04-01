<template>
  <div>
    <nav class="navbar navbar-dark bg-dark">
      <div class="container-fluid">
        <a class="navbar-brand" href="#">Perfil web</a>
      </div>
    </nav>
    <div class="container my-4">
      <h1>Lista de personas</h1>

    <form class="row g-3"  @submit.prevent="buscarPorId">
    <div class="col-auto">
      <input type="text" readonly class="form-control-plaintext" id="staticEmail2" value="Buscar por ID">
    </div>
    <div class="col-auto">
      <label for="inputID" class="visually-hidden">Password</label>
      <input type="txtId"  v-model="id"  class="form-control" id="txtId" placeholder="buscar por ID">
    </div>
    <div class="col-auto">
      <button type="submit" class="btn btn-primary mb-3" @click="buscarPorId()">Buscar</button>
    </div>
  </form>

        <table class="table table-bordered">
        <thead>
          <tr>
            <th>Id</th>
            <th>Nombre</th>
            <th>telefono</th>
            <th>cargo</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="pr in personas" :key="pr.id">
            <td>{{ pr.id }}</td>
            <td>{{ pr.nombre }}</td>
            <td>{{ pr.telefono }}</td>
            <td>{{ pr.cargo ? pr.cargo.descripcion : 'Sin cargo' }}</td>
            <td>
              <button class="btn btn-primary" @click="editarPersona(pr)">Editar</button>
              <button class="btn btn-danger" @click="eliminarPersona(pr)">Eliminar</button>
            </td>
          </tr>
        </tbody>
      </table>

      
      <div v-if="mostrarFormulario">
        <h2 v-if="editando">Editar persona</h2>
        <h2 v-else>Agregar persona</h2>
        <form @submit.prevent="guardarPersona">
          <input type="hidden" name="id" v-model="formulario.id">
          <input type="hidden" name="estado" v-model="formulario.estado">

          <div class="mb-3">
            <label for="nombre" class="form-label">Nombre:</label>
            <input type="text" class="form-control" id="nombre" v-model="formulario.nombre">
          </div>
          <div class="mb-3">
            <label for="telefono" class="form-label">Telefono:</label>
            <input type="text" class="form-control" id="telefono" v-model="formulario.telefono">
          </div>
          <div class="mb-3">
            <label for="cargo" class="form-label">Cargo:</label>
            <select class="form-select" id="cargo" v-model="formulario.idcargo">
              <option v-for="cargo in cargos" :key="cargo.id" :value="cargo.id">{{ cargo.descripcion }}</option>
            </select>
          </div>
          <button type="submit" class="btn btn-primary">{{ editando ? 'Actualizar' : 'Guardar' }}</button>
          <button type="button" class="btn btn-secondary" @click="cancelarEdicion">Cancelar</button>
        </form>
      </div>
      <button class="btn btn-primary mt-4" @click="mostrarFormulario = true; editando = false; formulario = { nombre: '',telefono: '', idcargo: 0 }">Agregar persona</button>
    </div>
  </div>
</template>


<script>
export default {
  name: 'App',
  data() {
    return {
      personas: [],
      persona: null,
      cargos: [],
      mostrarFormulario: false,
      editando: false,
      formulario: {
        id:0,
        nombre: '',
        telefono: '',
        estado: true,
        idcargo: 0
      },
      personaseleccionado: null
    }
  },
  mounted() {
    this.obtenerpersonas();
    this.obtenerCargos();
  },
  methods: {
    obtenerpersonas() {
      fetch('https://localhost:7036/api/Personas/ObtenerTodoPersonas')
        .then(response => response.json())
        .then(data => this.personas = data);
    },
    obtenerCargos() {
      fetch('https://localhost:7036/api/Cargos/ObtenerTodoCargos')
        .then(response => response.json())
        .then(data => this.cargos = data);
    },
    guardarPersona() {

      let personadata = {
        id: 0,
        nombre: '',
        idcargo: 0,
        telefono: 0,
        estado: true,
        cargo: {
          id: 0,
          descripcion: '',
          estado: true
        }
      }

      personadata.id = this.formulario.id;
      personadata.nombre = this.formulario.nombre;
      personadata.telefono = this.formulario.telefono;
      personadata.idcargo = this.formulario.idcargo;
      personadata.cargo = this.cargos.find(cargo => cargo.id === this.formulario.idcargo);
 

      if (this.editando) {
        fetch('https://localhost:7036/api/Personas/ActualizarPersona', {
          method: 'PUT',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(personadata)
        })
        .then(response => {
          if (response.status === 200) {
            return response.json();
          } else {
            throw new Error('Error al editar persona');
          }
        })
        .then(() => {
          this.obtenerpersonas();
          this.mostrarFormulario = false;
        })
        .catch(error => {
          console.error(error);
        });
      } else {
        fetch('https://localhost:7036/api/Personas/AgregarPersona', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(personadata)
      })
        .then(response => {
          if (response.status === 200) {
            return response.json();
          } else {
            throw new Error('Error al agregar persona');
          }
        })
        .then(() => {
          this.obtenerpersonas();
          this.mostrarFormulario = false;
        })
        .catch(error => {
          console.error(error);
        });
    
      }
    },
    eliminarPersona(persona) {
      if (confirm('¿Está seguro de que desea eliminar al empleado ' + persona.nombre + '?')) {
        fetch('https://localhost:7036/api/Personas/EliminarPersona' + '?id=' + persona.id, {
          method: 'DELETE'
        })
          .then(() => {
            this.obtenerpersonas();
          });
      }
    },
    editarPersona(persona) {
      this.editando = true;
      this.personaseleccionado = persona;
      this.formulario.id = persona.id;
      this.formulario.nombre = persona.nombre;
      this.formulario.telefono = persona.telefono;
      this.formulario.idcargo = persona.cargo.id;
      this.mostrarFormulario = true;
    },
    cancelarEdicion() {
      this.editando = false;
      this.personaseleccionado = null;
      this.formulario.nombre = '';
      this.formulario.telefono = '';
      this.formulario.idcargo = 0;
      this.mostrarFormulario = false;
    },
    async buscarPorId() {
      try {
        const response = await fetch(`https://localhost:7036/api/Personas/ObtenerPersonaxID/${this.id}`);
        const data = await response.json();
        if (response.status !== 200) {
          throw new Error(data);
        }
        this.personas = [data]; // Asignamos el resultado a la lista de personas
      } catch (error) {
        console.error(error);
        this.personas = []; // Limpiamos la lista de personas
      }
    }


  }
}
</script>
