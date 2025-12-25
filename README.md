
# 🔧 Troia – Delete Butonu

 **Canias ERP** üzerinde **TROIA** dili kullanılarak geliştirilmiş bir ekran da seçilmiş tüm satırları tablodan silen DELETE butonu Click Kodunu paylaşıyorum.  

------------------------------------------------------------
	
	MESSAGE CLK C0 WITH 'Seçilen Kayıtlar Listeden Çıkarılacaktır, devam edilsin mi?';
	
	IF CONFIRM == 'NO' THEN
		RETURN;
	ENDIF;
	
	LOCAL : INTEGER RN;
	RN = SECONDTABLE_ROWCOUNT;
	
	WHILE RN > 0 
	BEGIN
		READ SECONDTABLE WITH INDEX RN;
		RN = RN - 1;

	IF SECONDTABLE_SELECTED == 1 THEN
		CLEAR ROW SECONDTABLE;
	ENDIF;

	ENDWHILE;

------------------------------------------------------------

