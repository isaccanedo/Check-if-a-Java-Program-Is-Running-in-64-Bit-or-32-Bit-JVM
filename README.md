## JNI

Verifique se um programa Java está sendo executado em JVM de 64 bits ou 32 bits

# 1. Visão Geral
Embora o Java seja independente de plataforma, às vezes precisamos usar bibliotecas nativas. Nesses casos, podemos precisar identificar a plataforma subjacente e carregar as bibliotecas nativas apropriadas na inicialização.

Neste tutorial, aprenderemos diferentes maneiras de verificar se um programa Java está sendo executado em uma JVM de 64 ou 32 bits.

Primeiro, mostraremos como fazer isso usando a classe System.

Em seguida, veremos como usar a API Java Native Access (JNA) para verificar a quantidade de bits da JVM. JNA é uma biblioteca desenvolvida pela comunidade que permite todo o acesso nativo.

# 2. Usando a propriedade do sistema sun.arch.data.model
A classe System em Java fornece acesso a propriedades definidas externamente e variáveis de ambiente. Ele mantém um objeto Propriedades que descreve a configuração do ambiente de trabalho atual.

Podemos usar a propriedade de sistema “sun.arch.data.model” para identificar o número de bits JVM:

```
System.getProperty("sun.arch.data.model");
```

Ele contém “32” ou “64” para indicar uma JVM de 32 ou 64 bits, respectivamente. Embora essa abordagem seja fácil de usar, ela retorna “desconhecido” se a propriedade não estiver presente. Portanto, ele funcionará apenas com as versões Oracle Java.

Vamos ver o código:

```
public class JVMBitVersion {
    public String getUsingSystemClass() {
        return System.getProperty("sun.arch.data.model") + "-bit";
    }
 
    //... other methods
}
```

Vamos verificar essa abordagem por meio de um teste de unidade:

```
@Test
public void whenUsingSystemClass_thenOutputIsAsExpected() {
    if ("64".equals(System.getProperty("sun.arch.data.model"))) {
        assertEquals("64-bit", jvmVersion.getUsingSystemClass());
    } else if ("32".equals(System.getProperty("sun.arch.data.model"))) {
        assertEquals("32-bit", jvmVersion.getUsingSystemClass());
    }
}
```

# 3. Usando a API JNA

JNA (Java Native Access) oferece suporte a várias plataformas, como macOS, Microsoft Windows, Solaris, GNU e Linux.

Ele usa funções nativas para carregar uma biblioteca por nome e recuperar um ponteiro para uma função dentro dessa biblioteca.

### 3.1. Classe Nativa
Podemos usar POINTER_SIZE da classe Native. Esta constante especifica o tamanho (em bytes) de um ponteiro nativo na plataforma atual.

Um valor de 4 indica um ponteiro nativo de 32 bits, enquanto um valor de 8 indica um ponteiro nativo de 64 bits:

```
if (com.sun.jna.Native.POINTER_SIZE == 4) {
    // 32-bit
} else if (com.sun.jna.Native.POINTER_SIZE == 8) {
    // 64-bit
}
```

### 3.2. Classe de plataforma
Como alternativa, podemos usar a classe Platform, que fornece informações simplificadas da plataforma.

Ele contém o método is64Bit() que detecta se a JVM é de 64 bits ou não.

Vamos ver como ele identifica o bit:

```
public static final boolean is64Bit() {
    String model = System.getProperty("sun.arch.data.model",
                                      System.getProperty("com.ibm.vm.bitmode"));
    if (model != null) {
        return "64".equals(model);
    }
    if ("x86-64".equals(ARCH)
        || "ia64".equals(ARCH)
        || "ppc64".equals(ARCH) || "ppc64le".equals(ARCH)
        || "sparcv9".equals(ARCH)
        || "mips64".equals(ARCH) || "mips64el".equals(ARCH)
        || "amd64".equals(ARCH)
        || "aarch64".equals(ARCH)) {
        return true;
    }
    return Native.POINTER_SIZE == 8;
}
```

Aqui, a constante ARCH é derivada da propriedade “os.arch” por meio da classe System. É usado para obter a arquitetura do sistema operacional:

```
ARCH = getCanonicalArchitecture(System.getProperty("os.arch"), osType);
```

Essa abordagem funciona para diferentes sistemas operacionais e também com diferentes fornecedores de JDK. Portanto, é mais confiável do que a propriedade de sistema “sun.arch.data.model”.

# 4. Conclusão

Neste tutorial, aprendemos como verificar a versão de bits da JVM. Também observamos como JNA simplificou a solução para nós em diferentes plataformas.