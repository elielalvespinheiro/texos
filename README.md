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
