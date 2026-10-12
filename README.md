# banco.py
# Sistema bancário orientado a objetos
# Demonstra classes abstratas, herança, encapsulamento, polimorfismo e dunder methods

from abc import ABC, abstractmethod


# ============================================================
# CLASSE ABSTRATA
# ============================================================

class Conta(ABC):
    """
    Classe abstrata que representa uma conta bancária genérica.
    Não pode ser instanciada diretamente.
    """

    def __init__(self, titular, numero, saldo_inicial=0.0):
        self.titular = titular
        self.numero = numero
        self._saldo = 0.0
        self.saldo = saldo_inicial  # usa o setter com validação

    # ---------- PROPRIEDADES COM VALIDAÇÃO ----------

    @property
    def titular(self):
        """Retorna o nome do titular."""
        return self._titular

    @titular.setter
    def titular(self, valor):
        if not isinstance(valor, str) or len(valor.strip()) < 3:
            raise ValueError("O titular deve ter pelo menos 3 caracteres.")
        self._titular = valor.strip().title()

    @property
    def numero(self):
        """Retorna o número da conta."""
        return self._numero

    @numero.setter
    def numero(self, valor):
        if not isinstance(valor, str) or not valor.isdigit():
            raise ValueError("O número da conta deve conter apenas dígitos.")
        self._numero = valor

    @property
    def saldo(self):
        """Retorna o saldo atual."""
        return self._saldo

    @saldo.setter
    def saldo(self, valor):
        if not isinstance(valor, (int, float)):
            raise ValueError("O saldo deve ser um número.")
        if valor < 0:
            raise ValueError("O saldo não pode ser negativo.")
        self._saldo = float(valor)

    # ---------- MÉTODOS CONCRETOS ----------

    def depositar(self, valor):
        """Deposita um valor positivo na conta."""
        if valor <= 0:
            raise ValueError("O valor do depósito deve ser positivo.")
        self._saldo += valor
        return self._saldo

    def sacar(self, valor):
        """Saca um valor, desde que haja saldo suficiente."""
        if valor <= 0:
            raise ValueError("O valor do saque deve ser positivo.")
        if valor > self._saldo:
            raise ValueError("Saldo insuficiente para o saque.")
        self._saldo -= valor
        return self._saldo

    # ---------- MÉTODO ABSTRATO (POLIMORFISMO) ----------

    @abstractmethod
    def calcular_tarifa(self):
        """
        Retorna a tarifa mensal da conta.
        Cada tipo de conta implementa de forma diferente.
        """
        pass

    # ---------- DUNDER METHODS ----------

    def __str__(self):
        return (
            f"Conta {self.numero} | Titular: {self.titular} | "
            f"Saldo: R$ {self.saldo:.2f}"
        )

    def __repr__(self):
        return (
            f"{self.__class__.__name__}(titular={self.titular!r}, "
            f"numero={self.numero!r}, saldo={self.saldo!r})"
        )

    def __eq__(self, outro):
        if not isinstance(outro, Conta):
            return NotImplemented
        return self.numero == outro.numero

    def __lt__(self, outro):
        if not isinstance(outro, Conta):
            return NotImplemented
        return self.saldo < outro.saldo


# ============================================================
# CLASSES FILHAS (HERANÇA)
# ============================================================

class ContaCorrente(Conta):
    """Conta corrente com tarifa mensal fixa e limite de cheque especial."""

    def __init__(self, titular, numero, saldo_inicial=0.0, limite=500.0):
        super().__init__(titular, numero, saldo_inicial)
        self.limite = limite

    @property
    def limite(self):
        return self._limite

    @limite.setter
    def limite(self, valor):
        if valor < 0:
            raise ValueError("O limite não pode ser negativo.")
        self._limite = float(valor)

    def calcular_tarifa(self):
        """Conta corrente tem tarifa fixa de R$ 12,00."""
        return 12.00

    def __str__(self):
        return (
            f"[Conta Corrente] {super().__str__()} | "
            f"Limite: R$ {self.limite:.2f} | Tarifa: R$ {self.calcular_tarifa():.2f}"
        )


class ContaPoupanca(Conta):
    """Conta poupança com rendimento mensal e sem tarifa."""

    def __init__(self, titular, numero, saldo_inicial=0.0, rendimento=0.005):
        super().__init__(titular, numero, saldo_inicial)
        self.rendimento = rendimento

    @property
    def rendimento(self):
        return self._rendimento

    @rendimento.setter
    def rendimento(self, valor):
        if valor < 0 or valor > 0.1:
            raise ValueError("O rendimento deve estar entre 0 e 0.1 (0% a 10%).")
        self._rendimento = float(valor)

    def calcular_tarifa(self):
        """Conta poupança não tem tarifa."""
        return 0.00

    def aplicar_rendimento(self):
        """Aplica o rendimento mensal ao saldo."""
        self._saldo += self._saldo * self.rendimento
        return self._saldo

    def __str__(self):
        return (
            f"[Conta Poupança] {super().__str__()} | "
            f"Rendimento: {self.rendimento * 100:.2f}% | "
            f"Tarifa: R$ {self.calcular_tarifa():.2f}"
        )


class ContaPremium(Conta):
    """Conta premium com tarifa alta e cashback de 1% em depósitos."""

    def __init__(self, titular, numero, saldo_inicial=0.0, cashback=0.01):
        super().__init__(titular, numero, saldo_inicial)
        self.cashback = cashback

    @property
    def cashback(self):
        return self._cashback

    @cashback.setter
    def cashback(self, valor):
        if valor < 0 or valor > 0.05:
            raise ValueError("O cashback deve estar entre 0 e 0.05 (0% a 5%).")
        self._cashback = float(valor)

    def depositar(self, valor):
        """Deposita e aplica cashback sobre o valor."""
        super().depositar(valor)
        bonus = valor * self.cashback
        self._saldo += bonus
        return self._saldo

    def calcular_tarifa(self):
        """Conta premium tem tarifa de R$ 29,90."""
        return 29.90

    def __str__(self):
        return (
            f"[Conta Premium] {super().__str__()} | "
            f"Cashback: {self.cashback * 100:.2f}% | "
            f"Tarifa: R$ {self.calcular_tarifa():.2f}"
        )


# ============================================================
# DEMONSTRAÇÃO E TESTES
# ============================================================

def main():
    print("=" * 60)
    print("        SISTEMA BANCÁRIO ORIENTADO A OBJETOS")
    print("=" * 60)

    # ---------- CRIAÇÃO DE OBJETOS ----------
    contas = [
        ContaCorrente("miqueias saraiva", "1001", 1500.00, 800.00),
        ContaCorrente("ana clara", "1002", 300.00),
        ContaPoupanca("bruno costa", "2001", 5000.00, 0.007),
        ContaPoupanca("carla dias", "2002", 1200.00),
        ContaPremium("diego lima", "3001", 10000.00, 0.02),
        ContaPremium("elisa moura", "3002", 2500.00),
        ContaCorrente("felipe andrade", "1003", 0.00),
        ContaPoupanca("gabriela reis", "2003", 800.00),
        ContaPremium("henrique souza", "3003", 15000.00),
        ContaCorrente("isabela pinto", "1004", 4500.00, 1000.00),
    ]

    # ---------- LISTAGEM INICIAL ----------
    print("\n>>> Contas cadastradas:")
    for conta in contas:
        print(f"  {conta}")

    # ---------- POLIMORFISMO: MESMA CHAMADA, RESPOSTAS DIFERENTES ----------
    print("\n>>> Tarifas mensais (polimorfismo):")
    for conta in contas:
        print(f"  {conta.titular:<20} -> R$ {conta.calcular_tarifa():.2f}")

    # ---------- OPERAÇÕES ----------
    print("\n>>> Operações:")
    try:
        contas[0].depositar(500)
        print(f"  Depósito de R$ 500,00 em {contas[0].titular} -> {contas[0].saldo:.2f}")

        contas[0].sacar(200)
        print(f"  Saque de R$ 200,00 em {contas[0].titular} -> {contas[0].saldo:.2f}")

        contas[2].aplicar_rendimento()
        print(f"  Rendimento aplicado em {contas[2].titular} -> {contas[2].saldo:.2f}")

        contas[4].depositar(1000)
        print(f"  Depósito com cashback em {contas[4].titular} -> {contas[4].saldo:.2f}")

    except ValueError as erro:
        print(f"  Erro na operação: {erro}")

    # ---------- TRATAMENTO DE EXCEÇÕES ----------
    print("\n>>> Testando validações (try/except):")

    tentativas_invalidas = [
        ("Criar conta com titular curto", lambda: ContaCorrente("ab", "9999")),
        ("Número de conta inválido", lambda: ContaPoupanca("Teste", "ABC")),
        ("Saldo negativo", lambda: ContaCorrente("Teste", "9999", -100)),
        ("Saque maior que saldo", lambda: contas[1].sacar(99999)),
        ("Depósito negativo", lambda: contas[1].depositar(-50)),
        ("Rendimento fora do intervalo", lambda: ContaPoupanca("Teste", "8888", 100, 0.5)),
    ]

    for descricao, operacao in tentativas_invalidas:
        try:
            operacao()
            print(f"  {descricao}: não gerou erro (verifique!)")
        except ValueError as erro:
            print(f"  {descricao}: {erro}")

    # ---------- ORDENAÇÃO (usa __lt__) ----------
    print("\n>>> Contas ordenadas por saldo (usa __lt__):")
    for conta in sorted(contas):
        print(f"  {conta.titular:<20} R$ {conta.saldo:>10.2f}")

    # ---------- IGUALDADE (usa __eq__) ----------
    print("\n>>> Teste de igualdade (usa __eq__):")
    print(f"  contas[0] == contas[1]? {contas[0] == contas[1]}")
    print(f"  contas[0] == contas[0]? {contas[0] == contas[0]}")

    # ---------- REPR ----------
    print("\n>>> Representação técnica (__repr__):")
    print(f"  {contas[0]!r}")

    # ---------- TENTANDO INSTANCIAR CLASSE ABSTRATA ----------
    print("\n>>> Tentando instanciar a classe abstrata Conta:")
    try:
        Conta("Teste", "0000")
    except TypeError as erro:
        print(f"  Erro esperado: {erro}")

    print("\n" + "=" * 60)
    print("              FIM DA DEMONSTRAÇÃO")
    print("=" * 60)


if __name__ == "__main__":
    main()
