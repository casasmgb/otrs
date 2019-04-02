#  Quitar o agregar codigo a curso
## proceso para quitar el codigo de preinscripcion a un curso del DIGA

solo los cursos de acuerso institucional pueden tener codigo, para verficar es suficiente la aplicacion del 
SIGA externo, el entorno de pruebas esta en la url:
  
  http://172.16.0.101:8103/SistemaCencap/PrincipalExterno
  
se busca el curso y se verifica que pida o no el codigo de preinscricion.

Para quitar el codigo crear el procedimiento almacenado del script **quitar_codigo_curso.sql**
y ejecutarlo de la siguiente manera:

  SELECT seguimiento_capacitacion.tmp_mantenimiento_quitar_codigo_curso('CE/LP-T403-882/2018')  
                                                                                 ^                          
                                                                           sigla del curso

para agregar un codigo a un curso existe un procedimiento almacenado solo hay que ejecutarlo

  select * from seguimiento_capacitacion.spr_gn_generar_codigo_acuerdo_institucional(1894); y proporcionar el 
                                                                                      ^
                                                                                 procur_codigo
