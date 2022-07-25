<template>
<div class="grid-container">

  <div class="formView">
    <form action="" class="form" @submit.prevent="formSubmit">

      <h1 class="formTitle" style="width: 100%; font-size: 35px; text-align: center;">Formulario</h1> 

      <div class="form-group">
        <input v-model="creditForm.valorPresente" type="text" name="valorPresente" required />
        <span class="highlight"></span>
        <span class="bar"></span>
        <label for="valorPresente">Valor Presente (P)</label>
      </div>

      <div class="input-group">
        <div class="form-group">
          <input v-model="creditForm.plazo" type="text" name="plazo" required />
          <span class="highlight"></span>
          <span class="bar"></span>
          <label for="plazo">Plazo (n)</label>
        </div>

        <select v-model="creditForm.tiempoPlazo" name="tiempoPlazo" id="tiempoPlazo" placeholder="Seleccione tiempo"  style="height: 25px; width:fit-content; transform: translateX(-75%);" required>
          <option disabled value="">Elija una:</option>
          <option value="Meses">Meses</option>
          <option value="Años">Años</option>
        </select>
      </div>

      <div class="input-group">
        <div class="form-group">
          <input v-model="creditForm.tasaInteres" type="text" name="tasaInteres" required />
          <span class="highlight"></span>
          <span class="bar"></span>
          <label for="tasaInteres">Tasa de interés (i) en porcentaje</label>
        </div>

        <select v-model="creditForm.interesTiempo" name="interesTiempo" id="interesTiempo" placeholder="Seleccione tiempo"  style="height: 25px; width:fit-content; transform: translateX(-75%);" required>
          <option disabled value="">Elija una:</option>
          <option value="Mensual">Mensual</option>
          <option value="Anual">Anual</option>
        </select>

        <p style="height: 25px; width:fit-content; transform: translateX(-65%);">capitalizable</p>

        <select v-model="creditForm.capitalizable" name="capitalizable" id="capitalizable" placeholder="Seleccione tiempo"  style="height: 25px; width:fit-content; transform: translateX(-40%);" required>
          <option disabled value="">Elija una:</option>
          <option value="Mensualmente">Mensualmente</option>
          <option value="Anualmente">Anualmente</option>
        </select>
      </div>

      <button class="b1" type="submit" style="margin: 0% 30% 3% 30%">Generar ciclo de vida del crédito</button>
    </form>
    <div>
      <text style="color:blue; font-size: 25px; text-align: center; width: 100%;">La Formula de la anualidad es:</text> <br>
      <img src="@/assets/formula.png" alt=""> <br>
      <text style="color:blue; font-size: 25px; text-align: center; width: 100%;" v-if="success">A = {{this.a}}</text> <br>
      <button @click="exportTable()" v-if="success" style="margin: 15px; padding: 5px;">Export table to excel</button>
      <text style="color:red; font-size: 25px; text-align: center; width: 100%;" >{{this.formError}}</text>
    </div>
    <div class="formula">
    </div>
  </div>

  <div class="lifeCycle">
    <table id="lifeCycleTable" ref="exp_table">
      <thead>
        <tr>
          <th>Periodo</th>
          <th>Saldo</th>
          <th>Interés</th>
          <th>IVA</th>
          <th>Abono a capital</th>
          <th>Pago Mensual</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>0</td>
          <td>{{this.P}}</td>
          <td>-</td>
          <td>-</td>
          <td>-</td>
          <td>-</td>
        </tr>
      </tbody>
    </table>
  </div>
</div>
</template>
<script>
  import 'table2excel'
  export default {
    data() {
      return {
        tableData: [],
        tableColumns: [],
        formError: "",
        creditForm: {
          valorPresente: "",
          plazo: "",
          tiempoPlazo: "",
          tasaInteres: "",
          interesTiempo: "",
          capitalizable: ""
        },
        a: 0,
        n: 0,
        i: 0,
        P: 0,
        success: false
      };
    },
    methods: {
      formSubmit() {
        this.P = parseFloat(this.creditForm.valorPresente);
        this.n = parseFloat(this.creditForm.plazo);
        this.i = parseFloat(this.creditForm.tasaInteres) / 100;
        if (isNaN(this.P) || isNaN(this.n) || isNaN(this.i)) {
          this.formError = "El formulario se llenó mal";
          this.erraseForm();
        } else {
          if (this.creditForm.capitalizable == "Mensualmente") {
            if (this.creditForm.tiempoPlazo == "Años") this.n *= 12;
            if (this.creditForm.interesTiempo == "Anual") this.i /= 12;
          } else {
            if (this.creditForm.tiempoPlazo == "Meses") this.n /= 12;
            if (this.creditForm.interesTiempo == "Mensual") this.i *= 12;
          }
          this.a = (this.P * this.i * 1.16) / (1 - (1 / ((1 + this.i * 1.16) ** this.n)));
          this.fillTable();
          this.success = true;
        }
      },
      fillTable() {
        let table = document.getElementById("lifeCycleTable");
        let saldo_anterior = this.P;
        for (let i = 1; i <= this.n; i++) {
          
          let interes = saldo_anterior * this.i;
          let iva = interes*.16;
          let abono = this.a - interes - iva;
          saldo_anterior -= abono;

          let rowInsert = table.insertRow(table.rows.length);

          rowInsert.insertCell(0).innerHTML = i;
          rowInsert.insertCell(1).innerHTML = saldo_anterior.toFixed(2);
          rowInsert.insertCell(2).innerHTML = (interes).toFixed(2);
          rowInsert.insertCell(3).innerHTML = (iva).toFixed(2);
          rowInsert.insertCell(4).innerHTML = (abono).toFixed(2);
          rowInsert.insertCell(5).innerHTML = (this.a).toFixed(2);
        }
        this.tableData = table.tHead.innerHTML;
        this.tableColumns = table.tBodies[0].innerHTML;
      },
      exportTable() {
        var table2excel = new Table2Excel();
        table2excel.export(document.querySelectorAll("#lifeCycleTable"));
      },
      erraseForm() {
        this.creditForm.valorPresente = "";
        this.creditForm.plazo = "";
        this.creditForm.tiempoPlazo = "";
        this.creditForm.tasaInteres = "";
        this.creditForm.interesTiempo = "";
        this.creditForm.capitalizable = "";
      }
    },
    components: { 

    }
  }
</script>

<style> 
.grid-container {
  display: grid;
  height: 100%;
  width: 100%;
  grid-template-columns: 50% 50%;
}
.grid-item {
  padding: 15%
}
.formView {
  width: 100%;
  min-width: 620px;
  padding: 20% 5% 20% 5%;
  display: flex;
  flex-direction: column;
  align-items:center;
}
.lifeCycle{
  display: flex;
  flex-direction: column;
  align-items:center;
  margin: 5% 10% 0% 10%;
}
table {
  width: 100%;
  border-collapse:separate;
  background: linear-gradient(-90deg,rgb(255, 54, 54), rgb(255, 206, 102));
  border-radius:25px 25px 0px 0px;
}

table tbody tr 
{
  background: white;
  border-bottom: 1px solid;
  text-align: center;
}
/* table tbody tr:nth-of-type(even)
{
    background-color: rgb(241, 241, 241);
}  */
table td, table th 
{
  padding-top: 12px;
  padding: 12px;
  border-spacing: 0px 15px;
}

form {
  border-style: solid;
  border-radius: 25px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
.form-group {
	position: relative;
	margin-bottom: 25px;
  padding-left: 15px;

}
.input-group{
  display: flex;
  justify-content: center;
  align-items: center;
}
input {
	display: block;
	font-size: 14pt;
	padding: 10px;
	border: none;
	border-bottom: 1px solid #ccc;
}

input:focus {
	outline: none;
}

label {
	position: absolute;
	top: 10px;
	left: 5px;
	color: #999;
	font-size: 14pt;
	font-weight: normal;
	pointer-events: none;
	transition: all 0.2s ease;
}

input:focus ~ label,
input:valid ~ label {
	top: -20px;
	font-size: 10pt;
	color: #5264AE;
}

.bar {
	display: block;
	position: relative;
	width: 320px;
}

.bar:before,
.bar:after {
	content: "";
	height: 2px;
	width: 0;
	bottom: 1px;
	position: absolute;
	background: #5264AE;
	transition: all 0.2s ease;
}

.bar:before {
	left: 50%;
}

.bar:after {
	right: 50%;
}

input:focus ~ .bar:before,
input:focus ~ .bar:after {
	width: 50%;
}
</style>
