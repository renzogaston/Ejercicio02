# Ejercicio02


namespace EXAMENFINAL.Models
{
    public class Seminario
    {
        public int CodigoSeminario {  get; set; }
        public string NombreCurso { get; set; }
        public string HorarioClase {  get; set; }
        public int Capacidad {  get; set; }
    }
}


namespace EXAMENFINAL.Models
{
    public class RegistroAsistencia
    {
        public int NumeroRegistro { get; set; }
        public int CodigoSeminario { get; set; }
        public int CodigoEstudiante { get; set; }
        public DateTime FechaRegistro { get; set; }
    }

}



using EXAMENFINAL.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Data;
using Microsoft.Data.SqlClient;
using System.Data;

namespace EXAMENFINAL.Controllers
{
    public class SeminarioController : Controller
    {
        private readonly IConfiguration _config;
        public SeminarioController(IConfiguration config)
        {
            _config = config;
        }
        private SqlConnection GetConnection()
        {
            return new SqlConnection(
                _config.GetConnectionString("DefaultConnection"));
        }
        public IActionResult Index()
        {
           List<Seminario>lista=new List<Seminario>();
            using (SqlConnection connection = GetConnection())
            {
                SqlCommand cmd = new SqlCommand("dbo.listarSeminario", connection);
                cmd.CommandType = CommandType.StoredProcedure;
                connection.Open();

                SqlDataReader dr = cmd.ExecuteReader();
                while (dr.Read())
                {
                    lista.Add(new Seminario
                    {
                        CodigoSeminario = (int)dr["CodigoSeminario"],
                        NombreCurso = dr["NombreCurso"].ToString(),
                        HorarioClase = dr["HorarioClase"].ToString(),
                        Capacidad = (int)dr["Capacidad"]
                    });
                }
            }
            return View(lista);
        }
        public IActionResult Registrar(int id)
        {
            Seminario s = new Seminario();

            using (SqlConnection con = GetConnection())
            {
                SqlCommand cmd = new SqlCommand("dbo.ObtenerSeminarioPorCodigo", con);
                cmd.CommandType = CommandType.StoredProcedure;
                cmd.Parameters.AddWithValue("@CodigoSeminario", id);

                con.Open();
                SqlDataReader dr = cmd.ExecuteReader();

                if (dr.Read())
                {
                    s.CodigoSeminario = (int)dr["CodigoSeminario"];
                    s.NombreCurso = dr["NombreCurso"].ToString();
                }
            }
            return View(s);
        }
        [HttpPost]
        public IActionResult Registrar(int CodigoSeminario, int CodigoEstudiante)
        {
            int numeroRegistro;

            using (SqlConnection con = GetConnection())
            {
                SqlCommand cmd = new SqlCommand("dbo.RegistrarAsistencia", con);
                cmd.CommandType = CommandType.StoredProcedure;

                cmd.Parameters.AddWithValue("@CodigoSeminario", CodigoSeminario);
                cmd.Parameters.AddWithValue("@CodigoEstudiante", CodigoEstudiante);

                SqlParameter output = new SqlParameter("@NumeroRegistro", SqlDbType.Int)
                {
                    Direction = ParameterDirection.Output
                };
                cmd.Parameters.Add(output);

                con.Open();
                cmd.ExecuteNonQuery();

                numeroRegistro = (int)output.Value;
            }

            TempData["Mensaje"] = "Registro exitoso. Número de registro: " + numeroRegistro;
            return RedirectToAction("Index");
        }


    }
    
}




@model List<EXAMENFINAL.Models.Seminario>

@{
    ViewData["Title"] = "Lista de Seminarios";
}

<h2>Seminarios Disponibles</h2>

<table class="table table-bordered table-striped">
    <thead>
        <tr>
            <th>Código</th>
            <th>Curso</th>
            <th>Horario</th>
            <th>Capacidad</th>
            <th>Acción</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var item in Model)
        {
            <tr>
                <td>@item.CodigoSeminario</td>
                <td>@item.NombreCurso</td>
                <td>@item.HorarioClase</td>
                <td>@item.Capacidad</td>
                <td>
                    <a asp-action="Registrar"
                       asp-route-id="@item.CodigoSeminario"
                       class="btn btn-primary btn-sm">
                        Registrar
                    </a>
                </td>
            </tr>
        }
    </tbody>
</table>



@model EXAMENFINAL.Models.Seminario

@{
    ViewData["Title"] = "Registro al Seminario";
}

<h2>Registro al Seminario</h2>

@if (ViewBag.Mensaje != null)
{
    <div class="alert alert-success">
        @ViewBag.Mensaje
    </div>
}

<form asp-action="Registrar" method="post">
    <div class="mb-3">
        <label class="form-label">Código del Seminario</label>
        <input type="text"
               name="CodigoSeminario"
               class="form-control"
               value="@Model.CodigoSeminario"
               readonly />
    </div>

    <div class="mb-3">
        <label class="form-label">Nombre del Curso</label>
        <input type="text"
               class="form-control"
               value="@Model.NombreCurso"
               readonly />
    </div>

    <div class="mb-3">
        <label class="form-label">Código del Estudiante</label>
        <input type="number"
               name="CodigoEstudiante"
               class="form-control"
               required />
    </div>

    <button type="submit" class="btn btn-success">
        Registrar
    </button>

    <a asp-action="Index" class="btn btn-secondary">
        Volver
    </a>
</form>










