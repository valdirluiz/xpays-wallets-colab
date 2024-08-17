### 1. **Configuração do Projeto**

#### a. Adicionar Dependência do Google Pay

No arquivo `build.gradle` do módulo (geralmente `app`), adicione a dependência do
Google Pay:

```groovy
dependencies {
    implementation "com.google.android.gms:play-services-wallet:18.1.0"
}
```

#### b. Permissões no AndroidManifest
Adicione as permissões necessárias no seu arquivo `AndroidManifest.xml`:
```xml
<uses-permission android:name="android.permission.INTERNET"/>
```


### 2. Criar o Método para Verificação de Suporte à Carteira:

Aqui está um exemplo de um método em Kotlin que verifica se o dispositivo tem suporte para Google Pay e retorna `false` em caso de erro:

```kotlin
import android.content.Context
import android.util.Log
import com.google.android.gms.tasks.Task
import com.google.android.gms.wallet.Wallet
import com.google.android.gms.wallet.IsReadyToPayRequest
import com.google.android.gms.wallet.WalletConstants

fun isGooglePayAvailable(context: Context, callback: (Boolean) -> Unit) {
    val paymentsClient = Wallet.getPaymentsClient(context, Wallet.WalletOptions.Builder()
        .setEnvironment(WalletConstants.ENVIRONMENT_TEST) // Alterar para ENVIRONMENT_PRODUCTION em produção
        .build())

    val request = IsReadyToPayRequest.newBuilder()
        .addAllowedPaymentMethod(WalletConstants.PAYMENT_METHOD_CARD)
        .addAllowedPaymentMethod(WalletConstants.PAYMENT_METHOD_TOKENIZED_CARD)
        .build()

    val task: Task<Boolean> = paymentsClient.isReadyToPay(request)

    task.addOnCompleteListener { taskResult ->
        if (taskResult.isSuccessful) {
            // O Google Pay está disponível
            callback(taskResult.result ?: false)
        } else {
            // Em caso de erro, retornar false
            Log.e("TAG", "Erro ao verificar a disponibilidade do Google Pay", taskResult.exception)
            callback(false)
        }
    }
}
```

#### Como Usar o Método na Sua Atividade

Você pode chamar o método `isGooglePayAvailable` e obter o resultado através de um callback:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    isGooglePayAvailable(this) { isAvailable ->
        if (isAvailable) {
            Log.d("TAG", "O dispositivo suporta Google Pay.")
        } else {
            Log.d("TAG", "Google Pay não está disponível no dispositivo.")
        }
    }
}
```



### 3. **Preparar a Interface de Usuário**
Crie uma interface simples, como um botão, para iniciar o processo de push
provisioning. Por exemplo, no seu layout XML:
```xml
<Button
android:id="@+id/provision_button"
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:text="Adicionar Cartão ao Google Pay"/>
```

### 4. **Codificação do Push Provisioning**
Agora, você irá codificar a atividade onde o Push Provisioning será realizado. Aqui está
um exemplo de como implementar isso em uma `Activity`.
```kotlin
import android.os.Bundle
import android.util.Log

import android.widget.Button
import androidx.appcompat.app.AppCompatActivity
import com.google.android.gms.wallet.PushProvisioningRequest
import com.google.android.gms.wallet.Wallet
import com.google.android.gms.wallet.WalletConstants

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val provisionButton: Button = findViewById(R.id.provision_button)
        provisionButton.setOnClickListener {
            startPushProvisioning()
        }
    }

    private fun startPushProvisioning() {
        // Este deve ser um cartão opaco recebido do backend, em Base64
            val opaquePaymentCardBase64 = "seu_cartao_opaco_base64"
            val networkId = "visa" // ID da rede, ex: "visa" ou "mastercard"
            val tokenServiceProviderId = "example_tsp_id" // ID do TSP
            // Criar PushProvisioningRequest
            val pushProvisioningRequest = PushProvisioningRequest.newBuilder()
                .setOpaquePaymentCard(opaquePaymentCardBase64)
                .setNetworkId(networkId)
                .setTokenServiceProviderId(tokenServiceProviderId)
                .build()


            // Iniciando o Push Provisioning
            val paymentsClient = Wallet.getPaymentsClient(this, Wallet.    WalletOptions.Builder()
                .setEnvironment(WalletConstants.ENVIRONMENT_TEST)
                .build())

            val task = paymentsClient.pushProvisioning(pushProvisioningRequest)

            task.addOnCompleteListener { taskResult ->
                if (taskResult.isSuccessful) {
                    // Cartão adicionado com sucesso
                    Log.d("TAG", "Push Provisioning success")
                } else {
                    // Falha ao adicionar cartão
                    Log.e("TAG", "Push Provisioning failed", taskResult.exception)
                }
            }
    }
}
```