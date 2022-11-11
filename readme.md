# App Send Mail 

Aplicação de envio de e-mail usando o paradigma de Orientação a Objetos por meio da bilbioteca PHPMailer e fazendo uso de tratamento de erros (try e catch).

Essa aplicação faz uso do serviço de **SMTP** do gmail.

## Front-end da Aplicação

Na parte do front da aplicação temos um formulário (HTML puro e na estilização fiz uso do bootstrap) para você preencher com o **destinário** (a quem você deseja enviar o e-mail), o **assunto** e a **mensagem** do e-mail.

Tendo como o arquivo **processa_envio.php** para receber os dados do front por meio do método POST. 

## Back-end da Aplicação

- No Back, criamos a classe Mensagem que irá representar a mensagem que receberemos do front.

```
class Mensagem
{

    private $para = null;
    private $assunto = null;
    private $mensagem = null;
    public $status = array('codigo_status' => null, 'descricao_status' => '');

    public function __get($attr)
    {
        return $this->$attr;
    }

    public function __set($attr, $valor)
    {
        $this->$attr = $valor;
    }

    public function mensagemValida()
    {
        //
    }
}
```

- Logo após, criamos a instancia ao objeto e pela variável do objeto recuperamos os dados pela Superglobal $_POST[].

```
$mensagem = new Mensagem();

$mensagem->__set('para', $_POST['para']);
$mensagem->__set('assunto', $_POST['assunto']);
$mensagem->__set('mensagem', $_POST['mensagem']);
```

- Após fazer a instancia ao objeto, criamos a função que irá validar se os dados da mensagem são válidos e devemos ou não dar continuidade a lógica da aplicação.

```
public function mensagemValida()
    {
        if (empty($this->para) || empty($this->assunto) || empty($this->mensagem)) {
            return false;
        }

        return true;
    }
```

- Agora, incluírei a biblioteca PHPMailer no projeto

```
require "./bibliotecas/PHPMailer/Exception.php";
require "./bibliotecas/PHPMailer/OAuth.php";
require "./bibliotecas/PHPMailer/PHPMailer.php";
require "./bibliotecas/PHPMailer/POP3.php";
require "./bibliotecas/PHPMailer/SMTP.php";

use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\Exception;
```

- Após a inclusão do PHPMailer, criamos a função que irá condicionar se a os dados da mensagem são válidos ou não.

Negativei o if, pois caso entre na condição imprimirá 'Mensagem não é válida' caso contrário ele dará continuídade ao script.

```
if (!$mensagem->mensagemValida()) {
    echo 'Mensagem não é Válida';
    //echo "<scritp>alert('Você não preencheu alguma informação ou tentou acesso direto.'); </script>";
    header('Location: index.php');
}
```

- Dando continuídade ao script e com o PHPMailer já configurado, ele seguirá com a lógica do projeto fazendo uso do tratamento de erros usando o try/catch. 

```
//Create an instance; passing `true` enables exceptions
$mail = new PHPMailer(true);

try {
    //Server settings
    $mail->SMTPDebug = false;                      //Enable verbose debug output
    $mail->isSMTP();                                            //Send using SMTP
    $mail->Host       = 'smtp.gmail.com';                     //Set the SMTP server to send through
    $mail->SMTPAuth   = true;                                   //Enable SMTP authentication
    $mail->Username   = '(seuemail)';                     //SMTP username
    $mail->Password   = '(suasenha)';                               //SMTP password
    $mail->SMTPSecure = 'tls';            //Enable implicit TLS encryption
    $mail->Port       = 587;                                    //TCP port to connect to; use 587 if you have set `SMTPSecure = PHPMailer::ENCRYPTION_STARTTLS`

    //Recipients
    $mail->setFrom('(seuemail)', 'Luan Remetente');
    $mail->addAddress($mensagem->__get('para'));     //Add a recipient
    //$mail->addReplyTo('info@example.com', 'Information');
    //$mail->addCC('cc@example.com');
    //$mail->addBCC('bcc@example.com');

    //Attachments
    //$mail->addAttachment('/var/tmp/file.tar.gz');         //Add attachments
    //$mail->addAttachment('/tmp/image.jpg', 'new.jpg');    //Optional name

    //Content
    $mail->isHTML(true);                                  //Set email format to HTML
    $mail->Subject = $mensagem->__get('assunto');
    $mail->Body    = $mensagem->__get('mensagem');
    $mail->AltBody = 'É necessário utilizar um client que suporte HTML para ter acesso total ao conteúdo dessa mensagem';

    $mail->send();

    $mensagem->status['codigo_status'] = 1;
    $mensagem->status['codigo_descricao'] = 'E-mail enviado com sucesso';
} catch (Exception $e) {
    $mensagem->status['codigo_status'] = 2;
    $mensagem->status['descricao_status'] = 'Não foi possível enviar este email! Por favor tente novamente mais tarde. Detalhes do erro: ' . $mail->ErrorInfo;
}
```

- Chegando na parte final do projeto, após o envio do e-mail criei uma página HTML para dar um feedback de sucesso do envio do e-mail. 😀

```
<html>
    
<head>
    <meta charset="utf-8" />
    <title>App Mail Send</title>

    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
</head>

<body>
    <div class="container">
        <div class="py-3 text-center">
            <img class="d-block mx-auto mb-2" src="logo.png" alt="" width="72" height="72">
            <h2>Send Mail</h2>
            <p class="lead">Seu app de envio de e-mails particular!</p>
        </div>
        <div class="row">
            <div class="col-md-12">
                <?php if ($mensagem->status['codigo_status'] == 1) { ?>

                    <div class="container">
                        <h1 class="display-4 text-success">Sucesso</h1>
                        <p><?= $mensagem->status['descricao_status'] ?></p>
                        <a href="index.php" class="btn btn-success btn-lg mt-5 text-white">Voltar</a>
                    </div>

                <?php } ?>

                <?php if ($mensagem->status['codigo_status'] == 2) { ?>

                    <div class="container">
                        <h1 class="display-4 text-danger">Ops!</h1>
                        <p><?= $mensagem->status['descricao_status'] ?></p>
                        <a href="index.php" class="btn btn-success btn-lg mt-5 text-white">Voltar</a>
                    </div>

                <?php } ?>
            </div>
        </div>
    </div>
</body>

</html>
```

