# bug-free-octo-enigma
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MinhaCriptomoeda {
    string public nome = "MinhaCriptomoeda";
    string public simbolo = "MCT";
    uint8 public casasDecimais = 18;
    uint256 public totalSupply;
    address public proprietario;
    
    mapping(address => uint256) public saldo;
    mapping(address => mapping(address => uint256)) public permissões;

    event Transferencia(address indexed remetente, address indexed destinatario, uint256 valor);
    event Aprovacao(address indexed dono, address indexed beneficiario, uint256 valor);

    modifier apenasProprietario() {
        require(msg.sender == proprietario, "Apenas o proprietário pode chamar essa função");
        _;
    }

    constructor(uint256 _initialSupply) {
        totalSupply = _initialSupply * 10 ** uint256(casasDecimais);
        saldo[msg.sender] = totalSupply;
        proprietario = msg.sender;
    }

    function transferir(address _destinatario, uint256 _valor) public returns (bool sucesso) {
        require(_destinatario != address(0), "Endereço inválido");
        require(saldo[msg.sender] >= _valor, "Saldo insuficiente");
        
        saldo[msg.sender] -= _valor;
        saldo[_destinatario] += _valor;
        
        emit Transferencia(msg.sender, _destinatario, _valor);
        return true;
    }

    function aprovar(address _beneficiario, uint256 _valor) public returns (bool sucesso) {
        permissões[msg.sender][_beneficiario] = _valor;
        
        emit Aprovacao(msg.sender, _beneficiario, _valor);
        return true;
    }

    function transferirDa(address _remetente, address _destinatario, uint256 _valor) public returns (bool sucesso) {
        require(_destinatario != address(0), "Endereço inválido");
        require(saldo[_remetente] >= _valor, "Saldo insuficiente");
        require(permissões[_remetente][msg.sender] >= _valor, "Sem permissão suficiente");
        
        saldo[_remetente] -= _valor;
        saldo[_destinatario] += _valor;
        permissões[_remetente][msg.sender] -= _valor;
        
        emit Transferencia(_remetente, _destinatario, _valor);
        return true;
    }

    function alterarProprietario(address novoProprietario) public apenasProprietario {
        require(novoProprietario != address(0), "Endereço inválido");
        proprietario = novoProprietario;
    }

    function aumentarTotalSupply(uint256 quantidade) public apenasProprietario {
        totalSupply += quantidade;
        saldo[proprietario] += quantidade;
    }

    // Função adicional para distribuir tokens inicialmente
    function distribuirTokens(address destinatario, uint256 quantidade) public apenasProprietario {
        require(destinatario != address(0), "Endereço inválido");
        require(quantidade <= totalSupply, "Quantidade excede o fornecimento total");

        saldo[proprietario] -= quantidade;
        saldo[destinatario] += quantidade;
        
        emit Transferencia(proprietario, destinatario, quantidade);
    }
}
