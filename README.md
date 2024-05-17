# texos

/*create database hotel;*/

select * from tbl_clientes;
select * from tbl_quartos;

create table tbl_clientes (
	clientes_ID int NOT NULL AUTO_INCREMENT,
	nome varchar(255),
	idade int(255),
	hospedagem varchar(255),
	mensalidade varchar(255),
	primary key (clientes_ID)
);

create table tbl_quartos (
	quartos_ID int NOT NULL AUTO_INCREMENT,
	numero varchar(255),
	andar int(255),
	nomeclatura varchar(255),
	clientes_ID int,
	primary key (quartos_ID),
	foreign key (clientes_ID) REFERENCES tbl_clientes(clientes_ID)
);

insert into tbl_clientes (nome, idade, hospedagem, mensalidade)
values (
	"Jucimar Alencar",
	42,
	"síndico",
	NULL
);

insert into tbl_quartos (numero, andar, nomeclatura, clientes_ID)
values (
	"34",
	3,
	"quarto do síndico",
	4
);

update tbl_quartos 
set andar = 2
where quartos_ID = 2;

select tbl_clientes.nome, tbl_quartos.andar
from tbl_quartos
left join tbl_clientes on tbl_clientes.clientes_ID = tbl_quartos.quartos_ID;



** ATUALIZADOS **



select * from tbl_clientes;
select * from tbl_quartos;
select * from tbl_garagem;

/*
create table tbl_garagem (
	garagem_ID int auto_increment not null,
	numero_da_garagem int,
	cliente_ID int,
	primary key (garagem_ID),
	foreign key (cliente_ID) references tbl_clientes(clientes_ID)
);*/



insert into tbl_garagem (numero_da_garagem, cliente_ID)
values
(10, 4);

select tbl_clientes.nome, hospedagem, tbl_quartos.numero, andar, from tbl_clientes
left join tbl_quartos on tbl_clientes.clientes_ID = tbl_quartos.quartos_ID;

/*aqui estamos fazendo a busca por três tabelas, veja que alguns itens não têm relação, ou seja voltam null*/
SELECT *
FROM tbl_clientes
left JOIN tbl_quartos
ON tbl_clientes.clientes_ID = tbl_quartos.quartos_ID
AND tbl_quartos.andar = tbl_clientes.hospedagem
left JOIN tbl_garagem
ON tbl_clientes.clientes_ID = tbl_garagem.cliente_ID;

/**/

select * from tbl_clientes 
left join tbl_quartos 
on tbl_clientes.clientes_ID = tbl_quartos.quartos_ID 
and tbl_quartos.andar = tbl_clientes.hospedagem 
left join tbl_garagem  
on tbl_clientes.clientes_ID = tbl_garagem.garagem_ID;

/* motivo do erro é que o slq não entende o full outer join, acima esta uma buscar que funciona da mesma forma, 
 * basta usar o left e right join para acessar todos elementos das duas tabelas */
SELECT tbl_clientes.nome , tbl_quartos.numero
FROM tbl_clientes
FULL OUTER JOIN tbl_quartos ON  tbl_clientes.clientes_ID  = tbl_quartos.clientes_ID
ORDER BY tbl_clientes.nome;

DELETE from tbl_garagem  WHERE garagem_ID=4;

insert into tbl_clientes (nome, idade, hospedagem, mensalidade)
values (
	"Jacinto Matos",
	68,
	"mendigo",
	"só cobra"
);

insert into tbl_quartos (numero, andar, nomeclatura, clientes_ID)
values (
	"30", 
	3, 
	"suite",
	null
);
























































