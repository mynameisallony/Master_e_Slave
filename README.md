# PASSAR ARQUIVOS PARA OS SERVIDORES
scp -P 7979 -r nxt3000-redundancia nvxtubarao@192.168.9.xxx:~

# PREPARANDO REPOSITORIO
mv /home/nvxtubarao/nxt3000-redundancia/ /usr/src/
cd /usr/src/nxt3000-redundancia/programas

# INSTALANDO DEPENDENCIAS
rpm -ivh popt-devel-1.13-7.el6.x86_64.rpm

# PREPARANDO ARQUIVOS
tar -zxvf keepalived-1.2.2.tar.gz
cd keepalived-1.2.2

# INSTALACAO
./configure && make && make install
ln -s /usr/local/etc/rc.d/init.d/keepalived /etc/init.d/keepalived
ln -s /etc/init.d/keepalived /etc/rc3.d/S99keepalived
ln -s /etc/init.d/keepalived /etc/rc5.d/S99keepalived
ln -s /usr/local/etc/sysconfig/keepalived /etc/sysconfig/keepalived
ln -s /usr/local/sbin/keepalived /bin/keepalived

# ================================================================================================
# => ALTERACOES SOMENTE NO MASTER
# COMENTAR SCRIPT DE QUEDA
sed -i 's/\/scripts\/keepalived\/master-derrubar.sh/#\/scripts\/keepalived\/master-derrubar.sh/g' /scripts/keepalived/backup.sh
sed -i 's/\/etc\/init.d\/kibs stop/#\/etc\/init.d\/kibs stop/g' /scripts/keepalived/backup.sh
sed -i 's/\/scripts\/keepalived\/master-derrubar.sh/#\/scripts\/keepalived\/master-derrubar.sh/g' /scripts/keepalived/fault.sh
sed -i 's/\/etc\/init.d\/kibs stop/#\/etc\/init.d\/kibs stop/g' /scripts/keepalived/fault.sh
# ================================================================================================

# ================================================================================================
# => ALTERACOES SOMENTE NO SLAVE
cp /usr/src/nxt3000-update/patch_3.0.56/nxt/keepalived/slave/* /scripts/keepalived/
# REMOVER SCRIPTS NAO UTILIZADOS E COMENTAR SCRIPT PARA ASSUMIR
rm -f /scripts/keepalived/master-derrubar.sh
chmod 755 /scripts/keepalived/*
sed -i 's/\/scripts\/keepalived\/slave-assumir.sh/#\/scripts\/keepalived\/slave-assumir.sh/g' /scripts/keepalived/master.sh
# ================================================================================================

# CONFIGURACAO INICIAL
mkdir /etc/keepalived/
echo 'FAULT' > /etc/keepalived/status
cd /usr/src/nxt3000-redundancia/

# ================================================================================================
# ==> APENAS NO MASTER
cp configuracao/keepalived_master.txt /etc/keepalived/keepalived.conf
dos2unix /etc/keepalived/keepalived.conf
# ================================================================================================

# ================================================================================================
# ==> APENAS NO SLAVE
cp configuracao/keepalived_slave.txt /etc/keepalived/keepalived.conf
dos2unix /etc/keepalived/keepalived.conf
# ================================================================================================

# ALTERAR PARA O IP VIP DO CLIENTE
vim /etc/keepalived/keepalived.conf

# ALTERACAO NO ARQUIVO DO SERVICO
cp configuracao/service.keepalived /etc/init.d/keepalived

# REINICIA SERVICO
/etc/init.d/network restart
/etc/init.d/keepalived restart

# TESTAR
/etc/init.d/keepalived status
ip addr show eth0
tail -n 20 /var/log/messages | grep Keepalived
