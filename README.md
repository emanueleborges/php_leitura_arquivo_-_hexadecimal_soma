# php_leitura_arquivo_-_hexadecimal_soma
php_leitura_arquivo_+_hexadecimal_soma

 
  public function saveFile(){
    //$dir = 'C:\\xampp\\htdocs\\caminho\\'; // caminho path formato windows
	  $dir = '/var/www/html/caminho/'; // caminho path formato linux
    //mover arquivos para o servidor 
    foreach($_FILES as $file){
      move_uploaded_file($file['tmp_name'],$dir.$file["name"]);
      $ext = substr($file["name"],18,20); //pegar extensao
      if (trim($ext) == 'cer'){
        $openssl_cmd = 'openssl x509 -inform DER -in ' . $dir.$file["name"] . ' -pubkey -noout > '. $dir.substr($file["name"],0,18).'pem'; //gerar chave pública
        shell_exec( $openssl_cmd );
      }
    }
    $arquivo_0 = $arquivo_1 = $arquivo_2 = $contents0 = $contents1 = $contents2 = '';
    // Open a known directory, and proceed to read its contents
    if (is_dir($dir)) {
      if ($dh = opendir($dir)) {
        while (($file = readdir($dh)) !== false) {
            $files = scandir($dir);
            foreach ($files as $file){
              if (filetype($dir.$file)=='file'){
                if (file_exists($dir.$file)){
                $name_arq = (substr(str_replace("-","",$file),0,12)); 
                $extensao = (substr(str_replace("-","",$file),13,15));
                if ($extensao == 'prv' || $extensao == 'cer' || $extensao == 'pem' ){
                  $fp = fopen($dir.$file, "r");
                  $content = bin2hex(fread($fp, filesize($dir.$file)));
                  if ($extensao == 'cer'){
                    $arquivo_0 =  $name_arq; 
                    $contents0 =  $content; 
                  }
                  if ($extensao == 'prv'){
                    $arquivo_1 =  $name_arq; 
                    $contents1 =  $content; 
                  }
                  if ($extensao == 'pem'){
                    $arquivo_2      =  $name_arq; 
                    $pkey_details1  = openssl_pkey_get_details(openssl_pkey_get_public(file_get_contents($dir.$file))); //pegar chave publica
                    $key_pem1       = bin2hex($pkey_details1["rsa"]["n"]);
                    $contents_pub   = "30818902818100".$key_pem1."0203010001"; //conteudo PUB
                    $contents2      =  $contents_pub; 
                  }
                  if ( $arquivo_0 == $arquivo_1 && $arquivo_1 == $arquivo_2){
                    $valor = $arquivo_0;
                    $hexa0 = hexdec($valor);                         //transformar string para hexadecimal
                    $hexa1 = strtoupper(dechex($hexa0 + hexdec(1)));  //soma hexadecimal + 1
                    $hexa2 = strtoupper(dechex($hexa0 + hexdec(2)));  //soma hexadecimal + 2
                    $hexa3 = strtoupper(dechex($hexa0 + hexdec(3)));  //soma hexadecimal + 3
                    $hexa4 = strtoupper(dechex($hexa0 + hexdec(4)));  //soma hexadecimal + 4
                    $sql 	 = parent::sql("SELECT MAC_DOCSIS_WAN FROM TABELA WHERE CAMPO1 = '".$arquivo_0."'" );
                    $MAC_DOCSIS_WAN = $sql[0]->MAC_DOCSIS_WAN;
                    if (empty($CAMPO1) ||  $CAMPO1 == ''){
                      $sql = parent::sql("INSERT INTO TABELA (CAMPO1, CAMPO2, CAMPO3, CAMPO4, CAMPO5, CAMPO6, CAMPO7, CAMPO8, CAMPO9, CAMPO0) VALUES ('$arquivo_0','$hexa1','$hexa2','$hexa3','$hexa4', '$contents2', '$contents1', '$contents0', SYSDATE ,'0')");
                      //return json_encode($sql);
                    }
                    //$arquivo_0 = $arquivo_1 = $arquivo_2 = $contents0 = $contents1 = $contents2 = '';
                  }
                  fclose($fp);
                  unlink($dir.$file);
                }
              }
            }
          }
        }
        closedir($dh);
      }
    }
  }
