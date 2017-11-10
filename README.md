# formularz
formularz konkursowy
<html>
	<head>
		<link href="formularz2.css" type="text/css" rel="stylesheet" />
		
		<!-- Kodowanie strony -->
		<meta charset="utf-8" />

		<!-- Znaczniki meta pod SEO -->
		<meta name="description" content="Formularz aplikacyjny" />
		<meta name="keywords" content="formularz, konkurs" />
		<meta name="author" content="Irena" />

		<!-- Wspomaganie responsywności strony -->
		<meta name="viewport" content="width=device-width, initial-scale=1.0" />

		<title> Formularz aplikacyjny</title>

	</head>
	

	<head>
	<body>
		<h1 id="konkurs">Konkurs fotograficzny Bonteo</h1>
	
		<div id="formularz">
			<form action="formularz3.php" method="post" enctype="multipart/form-data">
				<!-- Formularz wyrównanie w tabeli -->
				<table>
					<tr>
						<th scope="row"><label>Imię:</label></th><td><input type="text" name="name" placeholder="Wpisz imię" /></td>
					</tr>
					<tr>
						<th scope="row"><label>Nazwisko:</label></th><td><input type="text" name="surname" placeholder="Wpisz nazwisko" /></td>
					</tr>
					<tr>
						<th scope="row"><label>E-mail:</label></th><td><input type="text" name="email" /></td>
					</tr>
					<tr>
						<th scope="row"><label>Telefon:</label></th><td><input type="text" name="phone" /></td>
					</tr>
					<tr>
						<td colspan="2">
							<textarea name="tekst">Dlaczego to Ty powinieneś wygrać nasz konkurs fotograficzny?</textarea>
						</td>
					</tr>
				</table>

				<!-- Formularz -->
				<div>
					<input type="radio" name="Pytanie1" value="Kobieta" checked /> Kobieta
					<input type="radio" name="Pytanie1" value="Mężczyzna" /> Mężczyzna
				</div>
				<div>
					<select id="kraj" name="wojewodztwo" multiple>
						<option value="wielkopolska">Wielkopolska</option>
						<option value="mazowsze">Mazowsze</option>
						<option value="małopolska">Małopolska</option>
						<option value="podlasie">Podlasie</option>
						<option value="lubuskie">Lubuskie</option>
						<option value="podkarpackie">Podkarpackie</option>
						<option value="kujawsko-pomorskie">Kujawsko-Pomorskie</option>
						<option value="warmińsko-mazurskie">Warmińsko-Mazurskie</option>
					</select>
				</div>
				<table>
					<tr>
						<th>
							<label>Gdzie i kiedy zostało wykonane zdjęcie?</label>
						</th>
					</tr> 
					<tr>
						<td><input type="text" name="question_start_work" /></td>
					</tr>
					<tr>
						<th><label>Jakiego sprzętu użyłeś/ łaś do wykonania zdjęcia?</label></th>
					</tr>
					<tr>
						<td><input type="text" name="question_pay" /></td>
					</tr>
				</table>
				<!-- Akceptacja regulaminu -->
				<div>
					<p>
						<input type="checkbox" name="regulamin" value="jeden" /> <a href="regulaminkonkursu.html">Akceptuję regulamin konkursu.</a>
					</p>
				</div>
				<div>
					<input type="file" name="photo" />
				</div>
				<!-- Wyślij -->
				<div id="submit">
					<input type="submit" value="Wyślij" />
				</div>
			</form>
		</div>
<?php
	if ($_SERVER['REQUEST_METHOD'] === "POST") {
		$okey = 1;
		$dir_for_photos = "dane/zdjecia/";
		$only_file_name = time() . basename($_FILES['photo'] ['name']);
		$file_name = $dir_for_photos . $only_file_name;
		//$sciezka = htmlspecialchars (trim($_POST['sciezka']));
		
		// sprawdzanie czy nic nie jest puste - jak jest - to blad
		if(empty($_POST["name"]) || empty($_POST["surname"]) || empty($_POST["email"]) || empty($_POST["phone"]) || empty($_POST["tekst"])) {
			$okey = 5;
		}
		else {//a@a.pl
			if(!preg_match("/^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,3}$/", trim($_POST["email"]))) {
				$okey = 6;
			}
			else {
				if(!preg_match("/^[0-9]{9,}$/", trim($_POST["phone"]))) {
					$okey = 7;
				}
				else {
					if(file_exists($file_name)) {
						$okey = 4;
					}
					else {
						$det = ' ;-;-; ';
						$data = trim($_POST['name']).$det.trim($_POST['surname']).$det.trim($_POST['email']).$det.trim($_POST['phone']).$det
							.trim($_POST["tekst"]).$det.$only_file_name.PHP_EOL;
							
						// *******************************************************************
						$plik = fopen("dane/users_data.txt", "a");
						fputs($plik, $data);
						fclose($plik);
						// *******************************************************************
						
						if($_FILES['photo'] ['size'] > 4000000) {
							$okey = 3;
						}
						else {
							$photo_type = pathinfo($file_name, PATHINFO_EXTENSION);
							
							if($photo_type != "png" && $photo_type != "jpg" && $photo_type != "jpeg" && $photo_type != "gif") {
								$okey = 2;
							}
							else {
								try {
									$pdo = new PDO('mysql:host=localhost;dbname=test;port=3306', 'test', 'test123');
									
									//echo "Połączenie udane! <br /><br />";
									
									
									
									$query = $pdo->prepare("INSERT INTO `test_konkurs` 
									(`Imie`, `Nazwisko`, `E-mail`, `Telefon`, `Pyt1`, `Pyt2`, `Pyt3`, `Plec`, `Region`, `Zdjecie`) 
									VALUES 
									(:imie, :nazwisko, :email, :telefon, :pyt1, :pyt2, :pyt3, :plec, :region, :sciezka)");
									
									if($query->execute(
									['sciezka' => $file_name, 'imie' => trim($_POST['name']), 'nazwisko' => trim($_POST['surname']),
									'email' => trim($_POST['email']), 'telefon' => trim($_POST['phone']), 'plec' => trim($_POST['Pytanie1']),
									'pyt1' => trim($_POST['tekst']), 'pyt2' => trim($_POST['question_pay']), 
									'pyt3' => trim($_POST['question_start_work']), 'region' => trim($_POST['wojewodztwo'])]
									)) {			
										//echo "Dodane!";
										
										if(move_uploaded_file($_FILES['photo']['tmp_name'], $file_name)) {
											$okey = 0;
										}
										else {
											$okey = 1;
										}
									}
									else {
										//echo "Błąd dodawania";
										$okey = 10;
									}
									
									
									//echo "<br /><br />";
								} catch(PDOException $e) {
									//echo "Błąd! ". $e->getMessage();;
									$okey = 11;
								}
							}
						}
					}
				}
			}
		}
		
		// reszta formularza
		if ($okey == 0) { // wszystko okey
		?>
			<h3>Dziękujemy za udział w konkursie!</h3>

			<p>Twoje odpowiedzi:</p>

			<ul>
			  <li>Imię: <b><?= trim($_POST["name"]); ?></b></li>
			  <li>Nazwisko: <b><?= trim($_POST["surname"]); ?></b></li>
			  <li>Email: <b><?= trim($_POST["email"]); ?></b></li>
			  <li>Telefon: <b><?= trim($_POST["phone"]); ?></b></li>
			</ul>
			
			<div>
				<img src="<?= $file_name; ?>" alt="" />
			</div>
		<?php
		} else {
			//echo "<p style=\"color:red\">Są błędy</p>";
			switch ($okey) {
				case 1:
					echo "zdjęcie nie mogło być załadowane.";
					break;
				case 2:
					echo "Nieodpowiedni format zdjęcia.";
					break;
				case 3:
					echo "Plik jest zbyt duży.";
					break;
				case 4: 
					echo "Plik nie istnieje.";
					break;
				case 5:
					echo "Uzupełnij wszystkie pola formularza.";
					break;
				case 6: 
					echo "Błędny adres e-mail.";
					break;
				case 7: 
					echo "Błędny numer telefonu.";
					break;
				default:
					echo "Nieznany błąd.";
			}
			

		}		
	}
  
  ?>
	
	</body>
	
</html>
