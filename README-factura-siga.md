# Desasociacion de facturas del sistema SIGA

## procedimiento para la desasociacion o desvinculacion correcta de facturas del sistema siga

todo el proceso se puede verificar en el entorde de desarrollo en la siguiente url

Siga externo
  
  http://172.16.0.101:8103/SistemaCencap/PrincipalExterno

Siga interno
  
  http://172.16.0.101:8102/SistemaCencap/

  usuari: 959197    
  password: 1-7

desde la aplicaion verificar que la persona que pago la factura este inscrita al curso del que se desea desvincular

una factura esta consolidada cuando esta se asocia o se vincula a una preinscripcion y conforman asi una inscripcion.

los scripts **desvincular_factura_seguimiento_capacitacion.sql*** y **desvincular_factura_acceso_externo.sql** realizaran 
todo el proceso de desvinculacion, solo se debe llamar al primero **desvincular_factura_seguimiento_capacitacion.sql** 
siguiendo el siguinete ejemplo:

  select seguimiento_capacitacion.spr_desvincular_factura('CE/LP-T347-737/2018', '5973672');
                                                                    ^                ^
                                                             sigla del curso         CI
                                                             
se envia la sigla del curso al que la persona esta inscrito y el CI de la persona.

la respuesa es una cadena con los pasos que realizo todo el procedimiento, el fac_codigo, perpre_codigo que son suficinetes 
para verificar que el cambio esta realizado con las sigientes consultas:

```sql
--perpre_codigo =20180913_NSSTDR
--fac_codigo = 428094
	
-- para verificar:
-- en esquema seguimineto_capacitacion
 SELECT * FROM seguimiento_capacitacion.facturas_persona_inscripcion f WHERE f.perpre_codigo IN ('20180913_NSSTDR' ); 			       -- delete
 select * from seguimiento_capacitacion.facturas where fac_codigo = 428094; 														                           -- update el saldo deve ser igual al saldo_total
 SELECT * FROM seguimiento_capacitacion.personas_inscripcion pe WHERE pe.perpre_codigo IN ( '20180913_NSSTDR' );		 			         -- delete	
-- en el esquema acceso exteno
 SELECT * from acceso_externo.cuenta_persona_inscripcion cpi WHERE cpi.perpre_codigo IN ( '20180913_NSSTDR' ) 					           -- delete
 select * from acceso_externo.persona_preinscripcion pe WHERE pe.perpre_codigo IN   ('20180913_NSSTDR') and pe.perpre_estado = 2;  -- update
 

 select * from seguimiento_capacitacion.historico_participantes
select * from acceso_externo.historico_participantes
```

tambien se puede verificar desde la aplicacion comprobando que la persona ya no este inscrita en el curso.
