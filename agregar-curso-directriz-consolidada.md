# Agregar curso a directriz consolidada

## procedimiento para agregar un curso a una directriz consolidada del SIGA.

para saber a que directriz agregar el curso buscar con los selects:
```
SELECT * FROM seguimiento_capacitacion.directrices AS d WHERE d.dir_codigo=34
select * from seguimiento_capacitacion.curso where dir_codigo = 34
```
una directriz puede tener varios cursos y apartir de estos curso se programan mas cursos que seran publicados.

la adicion de cursos se debe realizar considerando lo siguiente:
un curso tiene carga horaria
un curso tiene creditos
un curso tiene costo por hora
un curso tiene monto economico 
un curso tiene nivel de responsabilidad
un curso es de un tipo
 
estos para matros se pueden obtener de las parametricas de la base de datos.
 
para agregar un curso se debe segir el ejemplo del script **agregar_curso_a_directriz_consolidado.sql**
tener cuidado con los xml del final.

en el primero:
se definen las competencias basicas, si no se definira ninguno poner -1
 
    <combas_codigo>3</combas_codigo>
    
en el segundo:
se definen las competencias genericas, si no se definira ninguno poner -1
  
  <comgen_codigo>-1</comgen_codigo>   
  
en el tercero:
se definen las competencias especificas, si no se definira ninguno poner -1
  
  <comesp_codigo>-1</comesp_codigo>
  
despues de ejecutar la adicion se debe consolidar el curso con el select del final.
```
SELECT * FROM seguimiento_capacitacion.spr_upd_consolidar_cursos_malla_des(
	34 -- dir_codigo 
);
```

