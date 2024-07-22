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


Controller: 

@PostMapping("/tramitar/{siglaDocumento}/pessoa")
	public ResponseEntity<TramiteDocPessoaModel> tramitaDocumento(
			@PathVariable(name = "siglaDocumento") String siglaDocumento,
			@RequestBody TramiteDocPessoaModelInput tramiteDocPessoaModelInput) {
		Movement movement = movementService.criarMovimentacaoTramitarDocumentoPessoa(
														siglaDocumento,
														tramiteDocPessoaModelInput.getSubscritorId(),
														tramiteDocPessoaModelInput.getPessoaRecebedoraId());
		Mobil mobil = mobilService.buscarMobil(movement.getMobil().getMobilId());

		TramiteDocPessoaModel TramiteDocPessoaModel1 = movementModelMapper.toModelTramiteDocPessoa(movement);

		MobilModel mobilModel = mobilModelMapper.toModel(mobil);
		TramiteDocPessoaModel1.setMobilModel(mobilModel);
		return ResponseEntity.status(HttpStatus.OK).body(TramiteDocPessoaModel1);
	}

input:
package com.br.api.v1.model.input;

import lombok.Data;

@Data
public class TramiteDocPessoaModelInput {

    private Long subscritorId;
    private Long pessoaRecebedoraId;

}
model: package com.br.api.v1.model;

import com.br.domain.model.enums.TypeMovement;
import lombok.Getter;
import lombok.Setter;

import java.time.OffsetDateTime;

@Getter
@Setter
public class TramiteDocPessoaModel {

    private Long movementId;
    private TypeMovement typeMovement;
    private OffsetDateTime dataHoraCricao;
    private Long pessoaRecebedoraId;
    private Long subscritorId;
    private MobilModel mobilModel;

}

impl: 

package com.br.domain.service.impl;

import java.util.Optional;

import com.br.domain.exception.EntidadeNaoExisteException;
import com.br.domain.exception.MovimentacaoExistenteException;
import com.br.domain.model.Mobil;
import com.br.domain.model.enums.TipoMarca;
import com.br.domain.model.enums.TypeMovement;
import com.br.domain.repository.MobilRepository;
import com.br.domain.service.MobilService;
import com.br.infrastructure.external.service.user.UserFeignClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.stereotype.Service;
import com.br.domain.model.Movement;
import com.br.domain.repository.MovementRepository;
import com.br.domain.service.MovementService;

@Service
public class MovementServiceImpl implements MovementService {
	
	@Autowired
	MovementRepository movementRepository;

	@Autowired
	private MobilRepository mobilRepository;

	@Autowired
	private MobilService mobilService;

	@Autowired
	private UserFeignClient userFeignClient;

	@Autowired
	private MovementService movementService;

	@Override
	public Movement save(Movement movimentacao) {
	    return movementService.save(movimentacao);
	}

	public Movement verificarSePessoaRecebedoraAssinou(String siglaMobil, Long subscritorId, Long pessoaRecebedoraId) {
		Mobil mobil = buscarMobil(siglaMobil);
		//Percorrer todas movimentações do Mobil.
		for(Movement movement: mobil.getMovimentacoes()) {
			if((movement.getTypeMovement() == TypeMovement.TRAMITE_PARA_PESSOA) &&
					(movement.getSubscritorId() == subscritorId) &&
					(movement.getPessoaRecebedoraId() == pessoaRecebedoraId)) {
				return movement;
			}
		}
		return null;
	}

	@Override
	public Movement verificarSeOSubscritorAssinou(String siglaMobil, Long subscritorId) {
		Mobil mobil = buscarMobil(siglaMobil);
		for(Movement movement: mobil.getMovimentacoes()) {
			if((movement.getTypeMovement() == TypeMovement.ASSINATURA_COM_SENHA) &&
					(movement.getSubscritorId() == subscritorId)) {
				return movement;
			}
		}
		return null;
	}

	@Override
	public Movement criarMovimentacaoAssinarComSenha(String siglaMobil, Long subscritorId) {
		Movement movement = verificarSeOSubscritorAssinou(siglaMobil, subscritorId);

		if(movement != null) {
			throw new MovimentacaoExistenteException(movement.getMovementId());
		}

		Mobil mobil = buscarMobil(siglaMobil);
		mobilService.atribuirMarcaAoMobil(mobil, TipoMarca.ASSINAR_COM_SENHA);
		movement = criarMovimentacao(TypeMovement.ASSINATURA_COM_SENHA, subscritorId, null, mobil);
		mobilService.atualizarSiglaDoMobil(mobil);

		return movement;
	}

	@Override
	public Movement criarMovimentacaoTramitarDocumentoPessoa(String siglaMobil, Long pessoaRecebedoraId, Long subscritorId) {
		Movement movement = verificarSePessoaRecebedoraAssinou(siglaMobil, subscritorId, pessoaRecebedoraId);

		if(movement != null) {
			throw new MovimentacaoExistenteException(movement.getMovementId());
		}

		Mobil mobil = buscarMobil(siglaMobil);
		mobilService.atribuirMarcaAoMobil(mobil, TipoMarca.TRAMITAR_DOCUMENTO_PESSOA);
		return criarMovimentacao(TypeMovement.TRAMITE_PARA_PESSOA, subscritorId, null, mobil);
	}

	@Override
	public Movement findById(Long movimentacaoId) {
		Movement movement = movementService.findById(movimentacaoId);
		throw new EntidadeNaoExisteException("Movimentação não encontrado: " + movimentacaoId);
    }

	@Override
	public Page<Movement> findAll(Specification<Movement> spec, Pageable pageable) {
		return mobilRepository.findAll(spec, pageable);
	}

	@Override
	public Page<Movement> buscarMovimentacoesDoMobilFiltro(Long mobilId, Pageable pageable) {
		return movementService.buscarMovimentacoesDoMobilFiltro(mobilId, pageable);
	}

	@Override
	public Movement criarMovimentacao(TypeMovement typeMovement, Long subscritorId, Long pessoaRecebedoraId, Mobil mobil) {
		Movement movimentacao = new Movement();
		movimentacao.setSubscritorId(subscritorId);
		movimentacao.setPessoaRecebedoraId(pessoaRecebedoraId);
		movimentacao.setMobil(mobil);
		movimentacao.setTypeMovement(typeMovement);
		return movementService.save(movimentacao);
	}

	private Mobil buscarMobil(String siglaMobil) {
		mobilService.buscarMobil(siglaMobil);
				throw new EntidadeNaoExisteException("O Mobil (" + siglaMobil +") informado não existe.");
	}

}

service:

package com.br.domain.service;

import com.br.domain.model.Mobil;
import com.br.domain.model.Movement;
import com.br.domain.model.enums.TypeMovement;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.domain.Specification;

public interface MovementService {
	
	Movement save(Movement movimentacao);
	Movement verificarSeOSubscritorAssinou(String siglaDocumento, Long subscritorId);
	Movement criarMovimentacaoAssinarComSenha(String siglaDocumento, Long subscritorId);
	Movement findById(Long movimentacaoId);
	Movement criarMovimentacao(TypeMovement typeMovement, Long subscritorId, Long pessoaRecebedoraId, Mobil mobil);
	Page<Movement> findAll(Specification<Movement> spec, Pageable pageable);
	Page<Movement> buscarMovimentacoesDoMobilFiltro(Long mobilId, Pageable pageable);
	Movement criarMovimentacaoTramitarDocumentoPessoa(String siglaMobil, Long pessoaRecebedoraId, Long subscritorId);

}


*User Generater*

package com.br.core.config;

import java.security.SecureRandom;
import java.util.Random;
import java.util.UUID;

import org.springframework.stereotype.Component;

@Component
public class Generator {
	
	private static final String CARACTERES_UPPER = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    private static final String CARACTERES_LOWER = "abcdefghijklmnopqrstuvwxyz";
    private static final String DIGITOS = "0123456789";
    private static final String ESPECIAIS = "!@#$%^&*()-_=+[{]}|;:,<.>/?";

	private Long dado;
    private Long cont = 1L;
    private String num = "";

    public void sigla(UUID inicio) {
        if (inicio == null) {
            throw new NullPointerException("UUID cannot be null");
        }
        String uuidString = inicio.toString().replaceAll("-", "");
        Long inicioLong = Long.parseLong(uuidString.substring(0, 12)); // extrai os primeiros 12 caracteres do UUID como um Long
        this.dado = inicioLong;
        if (this.dado < 9) {
            Long can = cont + this.dado;
            num = "0000" + can;
        } else {
            Long can = cont + this.dado;
            num = "000" + can;
        }
    }

    public String getSigla(String nomeDepartamento) {
        return nomeDepartamento + this.num;
    }
    
    public static String password(int comprimento) {
        StringBuilder senha = new StringBuilder();
        Random random = new SecureRandom();

        // Adiciona pelo menos um caractere de cada tipo
        senha.append(pickRandom(CARACTERES_UPPER, random));
        senha.append(pickRandom(CARACTERES_LOWER, random));
        senha.append(pickRandom(DIGITOS, random));
        senha.append(pickRandom(ESPECIAIS, random));

        // Preenche o restante da senha com caracteres aleatórios
        for (int i = 4; i < comprimento; i++) {
            String conjuntoCaracteres = CARACTERES_UPPER + CARACTERES_LOWER + DIGITOS + ESPECIAIS;
            senha.append(pickRandom(conjuntoCaracteres, random));
        }

        // Embaralha a senha para torná-la mais aleatória
        char[] senhaArray = senha.toString().toCharArray();
        for (int i = 0; i < senhaArray.length; i++) {
            int j = random.nextInt(senhaArray.length);
            char temp = senhaArray[i];
            senhaArray[i] = senhaArray[j];
            senhaArray[j] = temp;
        }

        return new String(senhaArray);
    }

    private static char pickRandom(String conjuntoCaracteres, Random random) {
        int index = random.nextInt(conjuntoCaracteres.length());
        return conjuntoCaracteres.charAt(index);
    }
	
}
























































