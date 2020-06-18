<template>
    <section class="container mt-5">
        <h4 class="text-center">Listado de Libros</h4>
        <button v-b-modal.modal-agregar class="btn btn-primary my-auto btn-sm d-flex">Agregar</button>
        <table class="table mt-3">
            <thead class="thead-dark">
              <tr>
                <th scope="col">#</th>
                <th scope="col">Código</th>
                <th scope="col">Nombre</th>
                <th scope="col">Autor</th>
                <th scope="col">Editorial</th>
                <th scope="col">Publicación</th>
                <th scope="col">Acciones</th>
            </tr>
            </thead>
            <tbody>
                <tr v-for="(item, index) in libros" :key="index">
                    <td scope="row">{{item.id}}</td>
                    <td>{{item.codigo}}</td>
                    <td>{{item.nombre_libro}}</td>
                    <td>{{item.autor}}</td>
                    <td>{{item.editorial}}</td>
                    <td>{{item.publicacion}}</td>
                    <td>
                        <b-button class="btn-warning btn-sm mx-2" v-b-modal.modal-editar @click="editarLibro(item.id, index)">Editar</b-button>
                        <b-button class="btn-danger btn-sm mx-2" @click="eliminarLibro(item.id, index)">Eliminar</b-button>
                    </td>
                </tr>
            </tbody>
          </table>
        <div> 
            <!-- Agregar Libro -->
            <b-modal ref="modal-agregar"  id = "modal-agregar"  title = "Agregar Libro"> 
                <form>
                    <div class="form-group">
                        <label for="exampleInputEmail1" >Codigo</label>
                        <input type="text" class="form-control" v-model="libro.codigo" id="exampleInputEmail1" aria-describedby="emailHelp" placeholder="Codigo">
                    </div>
                    <div class="form-group">
                        <label for="exampleInputPassword1" >Nombre</label>
                        <input type="text" class="form-control" v-model="libro.nombre" id="exampleInputPassword1" placeholder="Nombre Libro">
                    </div>
                    <div class="form-group">
                        <select class="form-control" v-model="libro.autor">
                            <option v-for="item in autores" :key="item.id" :label="item.nombre" :value="item.id">Default select</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <select class="form-control" v-model="libro.editorial">
                            <option v-for="item in editoriales" :key="item.id" :label="item.nombre_editorial" :value="item.id">Default select</option>
                        </select>
                    </div>
                    <div class="form-group">
                         <label for="exampleInputPassword1">Año Publicacion</label>
                        <input type="text" class="form-control" v-model="libro.publicacion" id="exampleInputPassword1" placeholder="Año Publicación">
                    </div>
                    <button type="button" class="btn btn-primary mt-3" @click="agregarLibro(libro)">Agregar</button>
                </form>
            </b-modal > 

            <!-- Modal editar libro -->
            <b-modal ref="modal-editar"  id = "modal-editar"  title = "Editar Libro">
                <form>
                    <div class="form-group">
                        <label for="">Nombre</label>
                        <input type="text" class="form-control" v-model="libroEditar.nombre_libro">
                    </div>
                    <button type="button" @click="ActualizarLibro(libroEditar)">Actualizar</button>
                </form>
            </b-modal>
        </div >
        <b-alert
            :show="dismissCountDown"
            dismissible
            :variant="mensaje.color"
            @dismissed="dismissCountDown=0"
            @dismiss-count-down="countDownChanged"
        >
            {{mensaje.texto}}
        </b-alert>
    </section>
</template>

<script>
export default {
    components: {
        // modal: () => import('./modal')
    },
    data() {
        return {
            mensaje: {color: 'succes', texto: ''},
            dismissSecs: 5,
            dismissCountDown: 0,
            libros: [],
            libro: {codigo: '', nombre: '', autor: 0, editorial: 0, publicacion: ''},
            autores: [],
            editoriales: [],
            libroEditar: {}
       }
    },
    created() {
        this.showLibros()
        this.listarAutores()
        this.listarEditoriales()
    },
    mounted() {
        console.log('montado')
    },
    methods: {
        showLibros() {
            this.axios.get('/libros')
            .then(res => { 
                if (res.status == 200) {
                    this.libros = res.data
                    console.log(res.data)
                }
            })
            .catch( err => {
                console.log(err)
            })
        },
        agregarLibro(libro) {
            this.axios.post('/libros', libro)
            .then(res => {
                console.log(res.data)
                this.libros.push(res.data)
                this.libro.codigo = ''
                this.libro.nombre = ''
                this.libro.autor = 0
                this.libro.editorial = 0
                this.libro.publicacion = ''
                this.mensaje.color = 'success'
                this.mensaje.texto = 'Libro Agregado'
                this.showAlert()
                this.hideModal('modal-agregar')
            })
            .catch(e => {
                console.log(e)
                this.mensaje.color = 'danger'
                this.mensaje.texto = 'Libro no Agregado'
                this.showAlert()
                this.hideModal('modal-agregar')
            })
        },
        editarLibro(id) {
            // this.nombreEditado = this.libros[index].nombre_libro
            this.axios.get(`/libros/${id}`)
            .then(res => {
                this.libroEditar = res.data
            })
            .catch(e => {
                console.log(e.response);
            })
        },
        ActualizarLibro(item){
            this.axios.put(`/libros/${item.id}`, item)
                .then(res => {
                    let index = this.libros.findIndex( itemLibro => itemLibro.id === res.data.id);
                    this.libros[index].nombre_libro = res.data.nombre_libro;
                    this.mensaje.texto = 'Libro Actualizado'
                    this.mensaje.color = 'success'
                    this.showAlert();
                    this.hideModal('modal-editar')
            })
            .catch(e => {
                console.log(e);
                this.hideModal('modal-editar')
            })
            this.agregar = true;
        },
        eliminarLibro(id,index) {
            this.axios.delete(`/libros/${id}`)
            .then(res => {
                this.libros.splice(index, 1)
                this.mensaje.color = 'success'
                this.mensaje.texto = 'Libro Eliminado'
                this.showAlert()
            })
            .catch(error =>{
                console.log(error)
            })
        },
        listarAutores() {
            this.axios.get('/autores')
            .then(res => { 
                if (res.status == 200) {
                    this.autores = res.data
                    console.log(res.data)
                }
            })
            .catch( err => {
                console.log(err)
            })
        },
        listarEditoriales() {
            this.axios.get('/editoriales')
            .then(res => { 
                if (res.status == 200) {
                    this.editoriales = res.data
                    console.log(res.data)
                }
            })
            .catch( err => {
                console.log(err)
            })
        },
        countDownChanged(dismissCountDown) {
            this.dismissCountDown = dismissCountDown
        },
        showAlert() {
            this.dismissCountDown = this.dismissSecs
        },
        hideModal(modal) {
            this.$refs[`${modal}`].hide()
        }
    }
}
</script>

<style scoped>

</style>