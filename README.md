♻️ Hurda Kasaları Etiket Birleştirme Uygulaması

Bu proje, üretim sahasında oluşan hurda malzemelerin takibini daha verimli ve hatasız hale getirmek amacıyla geliştirilmiş bir uygulamadır.

Üretim sürecinde, farklı bölümlerden çıkan hurdalar aynı alaşım tipine göre gruplanarak tek bir hurda kasasında toplanmaktadır. Ancak her hurda, üretim onayı sırasında ayrı ayrı etiketlenmekte ve bu etiketler daha sonra aynı kasada bir araya gelmektedir. Bu kasalar, yeniden ergitilmek üzere ergitme fırınlarına gönderilmektedir.

Mevcut durumda, bir hurda kasası içinde birden fazla hurda etiketi bulunduğu için;

 * her bir etiketin kg ve miktar bilgilerinin tek tek kontrol edilmesi,

 * toplam tartım sonucu ile karşılaştırılması

gerekmekte ve bu süreç hem zaman kaybına hem de hata riskine yol açmaktadır.

Bu uygulama ile:

 * Aynı alaşıma sahip birden fazla hurda etiketi tek bir hurda kasasında birleştirilebilmekte,

 * Birleştirme işlemi sonucunda tek bir ana hurda etiketi oluşturularak kasa bu etiket üzerinden takip edilebilmektedir,

 * Tartım, karşılaştırma ve yeniden ergitme süreçleri daha hızlı, izlenebilir ve güvenilir hale getirilmektedir.

Uygulama, üretim sahasında operasyonel verimliliği artırmayı, manuel kontrol ihtiyacını azaltmayı ve hurda yönetim sürecini dijital olarak standartlaştırmayı hedeflemektedir.

Dialog Görseli Eklendi İnceleyebilirsiniz .

**** Ekranın Kullanım Açıklaması: 2 Kısım dan oluşmaktadır. 

İlk Kısım da Barkod okutulma alanına okutulan barkodlar ilk tabloya dolar burada birleştirmek istenilen tüm etiketler okutulmaktadır. Tablo doldurulduktan sonra 'Hesapla' Buttonu ile hesap makinesi mantığında tüm okutulan barkodların toplam miktar ve toplam kg bilgileri bulunup butonun yanındaki text alanlara bilgi gelir. Bu kuton sadece bir hesap makinesi olarak kullanılır .
Eğer etiketler okutulduktan sonra birleştirilmek isteniyorsa birleştirilmek istenen satırlar seçilir ve 'Birleştir ve Etiket Al' Butonuna basılır.
Bu buton seçilen etiketlerin aynı alaşım grubunda olması zorunluluğu bildirir ve sonrasında IASINVSTOCK tablosunda yeni açmış olduğumuz CLKHBATCHNUM alanı bağladığımız numaratör ile yeni bir kod alır ve bu alan updatelenir sonrasında izlenebilirlik ve raporlama yapabilmek için bu kurgu oluşturulmuştur.
En sonunda da verdiğimiz etiket şablonunu doldurur ve etiket basar.

İkinci Kısımda ise aynı şekilde bir barkod okutma alanı vardır bu alana okutulan herhangi bir barkod daha önce birleştirildi ise birleştirilmiş etiket kayıtları ikinci tabloyu doldurur. Birleştirme esnasında etiket alamama gibi bir teknik sebep olursa sonrasında buradan etiket alabilsinler ve etiketin içeriğindeki birleştirilmiş barkodları görebilmeleri için bu kısım yapılmıştır. İkinci kısımdaki 'Etiket Yazdır' Butonu sadece birleştirilmiş bu etiketi tekrardan yazdırma işlemi yapar .

Dilaog üzerinden event kodları şu şekildedir:

BEFORE.Click -> Burada Tablolarımızı oluşturup ve değişkenlerimizi tanımlıyoruz.

      SELECT '' AS MATERIAL, '' AS STEXT, '' AS BATCHNUM , 0.00 AS TOTALSTOCK ,
      	 '' AS QUNIT, 0.00 AS NETWEIGHT, 0.00 AS TOTALNETWEIGHT  , '' AS COMPONENT 
      	FROM IASBAS001 
      	WHERE 1=2 
      	INTO TBLKGINFO;
      
      SELECT '' AS MATERIAL, '' AS STEXT, '' AS BATCHNUM , 0.00 AS TOTALSTOCK ,
      	 '' AS QUNIT, 0.00 AS NETWEIGHT, 0.00 AS TOTALNETWEIGHT  , '' AS COMPONENT ,
      	 '' AS CLKHBATCHNUM 
      	FROM IASBAS001 
      	WHERE 1=2 
      	INTO TBLKGINFO1;
      
      
        
      OBJECT: 
       STRING MATMAT,
       STRING MATMAT2,
       STRING TMPYMINFO,
       DECIMAL TMPTOPALSTOCK,
       DECIMAL TMPTOTALKG,
       DECIMAL PSUMSTOCK,
       DECIMAL PSUMKG,
       NUMRANGE NUMRANGEREC,
       STRING TMPCLKHBATCHNUM,
       STRING STMPCLKHBATCHNUM,
       STRING STMPYMINFO,
       DECIMAL SPSUMSTOCK,
       DECIMAL SPSUMKG,
       INTEGER PARSECOUNT1,
       STRING PMAT1,
       STRING PCONF1,
       STRING MATMAT21,
       INTEGER PARSECOUNT,
       STRING PMAT,
       STRING PCONF;

 DELETE.Click -> üst tabloda satır silme işlemi için 

       MESSAGE CLK C0 WITH 'Seçilen Kayıtlar Listeden Çıkarılacaktır, devam edilsin mi?';
      
      IF CONFIRM == 'NO' THEN
      	RETURN;
      ENDIF;
      
      LOCAL : INTEGER RN;
      RN = TBLKGINFO_ROWCOUNT;
      
      WHILE RN > 0 
      BEGIN
      	READ TBLKGINFO WITH INDEX RN;
      	RN = RN - 1;
      
      	IF TBLKGINFO_SELECTED == 1 THEN
      		CLEAR ROW TBLKGINFO;
      	ENDIF;
      
      ENDWHILE;
      
      SUMSTOCK = 0.00;
      SUMKG = 0.00;

CANCEL.Click  -> Ekranı Kapatma Butonu
    
    SHUTDOWN;



OK.Click ->

    IF BARCODENO != '' THEN

	PARSECOUNT = 1;
	PMAT = '';
	PCONF = '';
	MATMAT2 = '';

	IF STRPOS(BARCODENO,'+') > 0 THEN
		BARCODENO = REPLACE(BARCODENO,'+','$');
	ENDIF;


	PARSE BARCODENO INTO PCONF DELIMITER '$' 
	BEGIN

		IF PARSECOUNT == 1 THEN
			PMAT = PCONF;
    /*MALZEME*/
    		ENDIF;
    
    
    		IF PARSECOUNT == 2 THEN
    			PCONF = PCONF;
    /*PARTİ*/
    		ENDIF;

		PARSECOUNT = PARSECOUNT + 1;
	ENDPARSE;

	BARCODENO = '';
	SELECT I.MATERIAL, X.STEXT, I.BATCHNUM, I.TOTALSTOCK,
		 I.QUNIT, B.NETWEIGHT, ( I.TOTALSTOCK * B.NETWEIGHT ) AS TOTALNETWEIGHT 
		FROM IASINVSTOCK I 
		INNER JOIN IASMATBASIC B ON I.CLIENT = B.CLIENT 
			AND I.COMPANY = B.COMPANY 
			AND I.MATERIAL = B.MATERIAL 
		INNER JOIN IASMATX  X ON B.CLIENT = X.CLIENT 
			AND B.COMPANY = X.COMPANY 
			AND I.PLANT = X.PLANT 
			AND B.MATERIAL = X.MATERIAL 
		WHERE I.CLIENT = SYS_CLIENT 
			AND I.COMPANY = '01' 
			AND I.MATERIAL = PMAT 
			AND I.BATCHNUM = PCONF 
			AND B.ISDELETE = 0 
			AND X.TEXTTYPE = 'M' 
			AND X.LANGU = SYS_LANGU 
			AND I.WAREHOUSE = 'YID' 
			AND I.STOCKPLACE = 'GNL' 
			AND I.SPECIALSTOCK = '9' 
			AND I.CLKHBATCHNUM = '' 
		INTO TMPINFO;


	IF SELECTED THEN
		LOCATERECORD SEQUENTIAL COLUMNS BATCHNUM , MATERIAL VALUES TMPINFO_BATCHNUM , TMPINFO_MATERIAL ON TBLKGINFO  ;

		IF SYS_STATUS == 1 THEN
			APPEND ROW TO TBLKGINFO ;
			SET TBLKGINFO TO RESIZE;
			MOVE-CORRESPONDING TMPINFO TO TBLKGINFO ;

			IF STRLEN(TBLKGINFO_MATERIAL) == 19 THEN
				MATMAT2 = REPLACE(TBLKGINFO_MATERIAL,STRSTR(TBLKGINFO_MATERIAL,16,3),'EPR');
			ELSE
				MATMAT2 = TBLKGINFO_MATERIAL + 'EPR';
			ENDIF;

			SELECT COMPONENT 
				FROM IASBOMITEM 
				WHERE CLIENT = SYS_CLIENT 
					AND COMPANY = '01' 
					AND ALTERNUM = '00' 
					AND MATERIAL = MATMAT2 
					AND COMPONENT LIKE 'YM%' 
				INTO TMPCOMP;

			MOVE TMPCOMP_COMPONENT TO TBLKGINFO_COMPONENT;
		ELSE
			MESSAGE CLK E0 WITH ' Bu Parti , Tabloya Okutuldu  !';
		ENDIF;

	ELSE
		MESSAGE CLK E0 WITH 'Etiket Bulunamadı';
	ENDIF;


	IF SUMSTOCK != 0.00 
			&& SUMKG != 0.00 THEN
		SUMSTOCK = 0.00;
		SUMKG = 0.00;
	ENDIF;

    ELSE
	

	PARSECOUNT1 = 1;
	PMAT1 = '';
	PCONF1 = '';
	MATMAT21 = '';

	IF TBLKGINFO1_ROWCOUNT > 0 THEN
		MESSAGE CLK E0 WITH 'Barkod Okutamazsınız Önce Tablodaki Etiketi Alınız!';
		RETURN;
	ENDIF;


	IF STRPOS(BARCODENO1,'+') > 0 THEN
		BARCODENO1 = REPLACE(BARCODENO1,'+','$');
	ENDIF;


	PARSE BARCODENO1 INTO PCONF1 DELIMITER '$' 
	BEGIN

    		IF PARSECOUNT1 == 1 THEN
    			PMAT1 = PCONF1;
    /*MALZEME*/
    		ENDIF;
    
    
    		IF PARSECOUNT1 == 2 THEN
    			PCONF1 = PCONF1;
    /*PARTİ*/
    		ENDIF;

		PARSECOUNT1 = PARSECOUNT1 + 1;
	ENDPARSE;

	BARCODENO1 = '';
	SELECT CLKHBATCHNUM 
		FROM IASINVSTOCK 
		WHERE CLIENT = SYS_CLIENT 
			AND COMPANY = '01' 
			AND MATERIAL = PMAT1 
			AND BATCHNUM = PCONF1 
			AND WAREHOUSE = 'YID' 
			AND STOCKPLACE = 'GNL' 
			AND SPECIALSTOCK = '9' 
			AND CLKHBATCHNUM != '' 
		INTO TMPHBATCH;


	IF NOTSELECTED THEN
		MESSAGE CLK E0 WITH 'Okutulan Barkod Birleştirilmemiştir.';
		RETURN;
	ENDIF;

	SELECT I.MATERIAL, X.STEXT, I.BATCHNUM, I.TOTALSTOCK,
		 I.QUNIT, B.NETWEIGHT, ( I.TOTALSTOCK * B.NETWEIGHT ) AS TOTALNETWEIGHT , '' AS COMPONENT,
		 I.CLKHBATCHNUM 
		FROM IASINVSTOCK I 
		INNER JOIN IASMATBASIC B ON I.CLIENT = B.CLIENT 
			AND I.COMPANY = B.COMPANY 
			AND I.MATERIAL = B.MATERIAL 
		INNER JOIN IASMATX  X ON B.CLIENT = X.CLIENT 
			AND B.COMPANY = X.COMPANY 
			AND I.PLANT = X.PLANT 
			AND B.MATERIAL = X.MATERIAL 
		WHERE I.CLIENT = SYS_CLIENT 
			AND I.COMPANY = '01' 
			AND B.ISDELETE = 0 
			AND X.TEXTTYPE = 'M' 
			AND X.LANGU = SYS_LANGU 
			AND I.WAREHOUSE = 'YID' 
			AND I.STOCKPLACE = 'GNL' 
			AND I.SPECIALSTOCK = '9' 
			AND I.CLKHBATCHNUM = TMPHBATCH_CLKHBATCHNUM 
		INTO TMPINFO1;


	LOOP AT TMPINFO1 
	BEGIN
		APPEND ROW TO TBLKGINFO1 ;
		SET TBLKGINFO1 TO RESIZE;
		MOVE-CORRESPONDING TMPINFO1 TO TBLKGINFO1 ;

		IF STRLEN(TBLKGINFO1_MATERIAL) == 19 THEN
			MATMAT21 = REPLACE(TBLKGINFO1_MATERIAL,STRSTR(TBLKGINFO1_MATERIAL,16,3),'EPR');
		ELSE
			MATMAT21 = TBLKGINFO1_MATERIAL + 'EPR';
		ENDIF;

		SELECT COMPONENT 
			FROM IASBOMITEM 
			WHERE CLIENT = SYS_CLIENT 
				AND COMPANY = '01' 
				AND ALTERNUM = '00' 
				AND MATERIAL = MATMAT21 
				AND COMPONENT LIKE 'YM%' 
			INTO TMPCOMP1;

		MOVE TMPCOMP1_COMPONENT TO TBLKGINFO1_COMPONENT;
	ENDLOOP;

    ENDIF;

BTNPRINTTOTAL.Click  -> Birleştir ve Etiket Al Butonu 

        TMPYMINFO = '';
        TMPTOPALSTOCK = 0.00;
        TMPTOTALKG = 0.00;
        PSUMSTOCK = 0.00;
        PSUMKG = 0.00;
        SELECT '' AS MATERIAL, '' AS STEXT, '' AS BATCHNUM , 0.00 AS TOTALSTOCK ,
        	 '' AS QUNIT, 0.00 AS NETWEIGHT, 0.00 AS TOTALNETWEIGHT  , '' AS COMPONENT 
        	FROM IASBAS001 
        	WHERE 1=2 
        	INTO TBLKGLIST;
        
        /*Bu tablo seçilen birleştirilecek hurda partilerinin listesini tutacak bu partilerin 
        	yeni bir numaratör den numara alıp IASINVSTOCK_CLKHBATCHNUM alanına güncellenecek en son etiket verecek */
        
        LOOP AT TBLKGINFO WHERE TBLKGINFO_SELECTED == 1 
        BEGIN
        	MATMAT = '';
        
        	IF STRLEN(TBLKGINFO_MATERIAL) == 19 THEN
        		MATMAT = REPLACE(TBLKGINFO_MATERIAL,STRSTR(TBLKGINFO_MATERIAL,16,3),'EPR');
        	ELSE
        		MATMAT = TBLKGINFO_MATERIAL + 'EPR';
        	ENDIF;
        
        	SELECT COMPONENT 
        		FROM IASBOMITEM 
        		WHERE CLIENT = SYS_CLIENT 
        			AND COMPANY = '01' 
        			AND ALTERNUM = '00' 
        			AND MATERIAL = MATMAT 
        			AND COMPONENT LIKE 'YM%' 
        		INTO TMPROWMAT;
        
        
        	IF TMPYMINFO == '' THEN
        		TMPYMINFO = TMPROWMAT_COMPONENT;
        	ELSE
        
        		IF TMPROWMAT_COMPONENT != TMPYMINFO THEN
        			MESSAGE CLK E0 WITH 'Malzemeler Aynı Alaşımdan Değil Kontrol Ediniz !';
        			RETURN 0;
        		ENDIF;
        
        	ENDIF;
        
        	TMPYMINFO = TMPROWMAT_COMPONENT;
        	TMPTOPALSTOCK = TMPTOPALSTOCK + TBLKGINFO_TOTALSTOCK;
        	TMPTOTALKG = TMPTOTALKG + TBLKGINFO_TOTALNETWEIGHT;
        	APPEND ROW TO TBLKGLIST;
        	MOVE-CORRESPONDING TBLKGINFO TO TBLKGLIST ;
        ENDLOOP;
        
        PSUMSTOCK = TMPTOPALSTOCK;
        PSUMKG = ROUND(TMPTOTALKG,2);
        /*Birleştirilen toplam hurda etiketlerinin IASINVSTOCK_CLKHBATCHNUM  updatelenmesi*/
        IF TBLKGLIST_ROWCOUNT > 0 THEN
        	TMPCLKHBATCHNUM =  NUMRANGEREC.NEWNUMBERS('01','INV-PRD-H',SYS_CURRENTDATE);
        
        	LOOP AT TBLKGLIST 
        	BEGIN
        		SELECT * 
        			FROM IASINVSTOCK 
        			WHERE CLIENT = SYS_CLIENT 
        				AND COMPANY = '01' 
        				AND WAREHOUSE = 'YID' 
        				AND STOCKPLACE = 'GNL' 
        				AND SPECIALSTOCK = '9' 
        				AND BATCHNUM = TBLKGLIST_BATCHNUM 
        				AND MATERIAL = TBLKGLIST_MATERIAL 
        				AND CLKHBATCHNUM = '' ;
        
        
        		IF SELECTED THEN
        			BEGINTRAN;
        			UPDATE IASINVSTOCK 
        				SET CLKHBATCHNUM = TMPCLKHBATCHNUM 
        				WHERE CLIENT = SYS_CLIENT 
        					AND COMPANY = '01' 
        					AND WAREHOUSE = 'YID' 
        					AND STOCKPLACE = 'GNL' 
        					AND SPECIALSTOCK = '9' 
        					AND BATCHNUM = TBLKGLIST_BATCHNUM 
        					AND MATERIAL = TBLKGLIST_MATERIAL 
        					AND CLKHBATCHNUM = '' ;
        
        
        			IF SYS_STATUS == 1 THEN
        				ROLLBACKTRAN;
        				RETURN 0;
        			ELSE
        				COMMITTRAN;
        			ENDIF;
        
        		ELSE
        			MESSAGE CLK E0 WITH 'Etiket Birleştirilmiştir Tekrar Birleştirilemez !';
        			RETURN 0;
        		ENDIF;
        
        	ENDLOOP;
        
        /*Tek Bir etiket ver*/
        	THIS.PRINTHURDABARCODE(TMPCLKHBATCHNUM,TMPYMINFO,PSUMSTOCK,PSUMKG);
        	LOCAL : INTEGER RN1;
        	RN1 = TBLKGINFO_ROWCOUNT;
        
        	WHILE RN1 > 0 
        	BEGIN
        		READ TBLKGINFO WITH INDEX RN1;
        		RN1 = RN1 - 1;
        
        		IF TBLKGINFO_SELECTED == 1 THEN
        			CLEAR ROW TBLKGINFO;
        		ENDIF;
        
        	ENDWHILE;
        
        	CLEAR ALL TBLKGLIST;
        ELSE
        	MESSAGE CLK I0 WITH 'Satır Seçiniz !!';
        ENDIF;
        
*** Kullanılan PRINTHURDABARCODE Methodu -->
      
      
      PARAMETERS: 
       STRING PTMPCLKHBATCHNUM,
       STRING PTMPYMINFO,
       DECIMAL PPSUMSTOCK,
       DECIMAL PPSUMKG;
      
      /*Tek Bir etiket ver*/
      OBJECT: 
       STRING RAWCMD,
       STRING USERIP,
       STRING USERINFO,
       STRING PRINTERPATH;
      
      RAWCMD = '';
      USERIP = '';
      USERINFO = '';
      PRINTERPATH = '';
      USERINFO = GETUSERINFO('client_address');
      PARSE USERINFO INTO USERIP DELIMITER '@' ;
      BEGIN
      USERIP=USERIP;
      ENDPARSE;
      
      
      IF  STRPOS(USERIP,'(') > 0 THEN
      	USERIP = STRSTR(USERIP,STRPOS(USERIP,'('),STRLEN(USERIP)-STRPOS(USERIP,'('));
      	USERIP = REPLACE(USERIP,')',''));
      	USERIP = REPLACE (USERIP,')','');
      	PRINTERPATH = '*\\'+USERIP+'\ZPRINTER';
      ELSE
      	USERIP = REPLACE (USERIP,')','');
      	PRINTERPATH = '*\\'+USERIP+'\ZPRINTER';
      ENDIF;
      
      SELECT BARCODELAYOUT 
      	FROM IASINV017 
      	WHERE CLIENT = SYS_CLIENT 
      		AND COMPANY = '01' 
      		AND BARCODETYPE = 'TPRD03' 
      	INTO TMPCLKIASINV017;
      
      RAWCMD = TMPCLKIASINV017_BARCODELAYOUT;
      RAWCMD = REPLACE(RAWCMD,'PHBATCH',PTMPCLKHBATCHNUM);
      RAWCMD = REPLACE(RAWCMD,'PROWMAT',PTMPYMINFO);
      RAWCMD = REPLACE(RAWCMD,'PBATCHNUM',PTMPCLKHBATCHNUM);
      RAWCMD = REPLACE(RAWCMD,'PUSER',SYS_USER);
      RAWCMD = REPLACE(RAWCMD,'PQUAN',PPSUMSTOCK);
      RAWCMD = REPLACE(RAWCMD,'PWEIGHT',PPSUMKG);
      RAWCMD = REPLACE(RAWCMD,'PDATETIME',SYS_CURRENTDATE);
      RAWCMD = REPLACE(RAWCMD,'PHTOTALKG',CEIL(PPSUMKG));
      PRINTTEXT RAWCMD TO PRINTERPATH CODEPAGE 'ISO-8859-9';


BTNPRINTTOTAL1.Click -> Etiket Yazdır Butonu 


      STMPCLKHBATCHNUM = '';
      STMPYMINFO = '';
      SPSUMSTOCK = 0.00;
      SPSUMKG = 0.00;
      
      LOOP AT TBLKGINFO1 WHERE TBLKGINFO1_ROWCOUNT > 0 
      BEGIN
      	STMPCLKHBATCHNUM = TBLKGINFO1_CLKHBATCHNUM;
      	STMPYMINFO = TBLKGINFO1_COMPONENT;
      	SPSUMSTOCK = SPSUMSTOCK + TBLKGINFO1_TOTALSTOCK;
      	SPSUMKG = SPSUMKG + TBLKGINFO1_TOTALNETWEIGHT;
      ENDLOOP;
      
      SPSUMKG = ROUND(SPSUMKG,2);
      THIS.PRINTHURDABARCODE(STMPCLKHBATCHNUM,STMPYMINFO,SPSUMSTOCK,SPSUMKG);
      
      CLEAR ALL TBLKGINFO1;


BTNHESAPLA.Click -> Hesapla Butonu 


    
    TMPTOPALSTOCK = 0.00;
    TMPTOTALKG = 0.00;
    SUMSTOCK = 0.00;
    SUMKG = 0.00;
    
    
    LOOP AT TBLKGINFO WHERE TBLKGINFO_ROWCOUNT > 0 
    BEGIN
    	TMPTOPALSTOCK = TMPTOPALSTOCK + TBLKGINFO_TOTALSTOCK;
    	TMPTOTALKG = TMPTOTALKG + TBLKGINFO_TOTALNETWEIGHT;
    ENDLOOP;
    
    SUMSTOCK = TMPTOPALSTOCK;
    SUMKG = TMPTOTALKG;
