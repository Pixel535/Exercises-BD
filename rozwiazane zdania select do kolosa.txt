---------------------------------

select imie_nazwisko, ilosc_zawodow
from
(select imie||' '||nazwisko as imie_nazwisko, count(nr_zawodow) as ilosc_zawodow
from bd3_zawodnicy z
            join bd3_wyniki w on z.nr_zawodnika = w.nr_zawodnika
            group by imie||' '||nazwisko
            having count(lokata_w_biegu) > 2
            order by ilosc_zawodow desc)
where rownum < 11
;


----------------------------------------

select imie_nazwisko, plec, data_urodzenia, nazwa_klubu, laczna_ilosc_punktow
from
(select imie||' '||nazwisko as imie_nazwisko, plec, data_urodzenia, nazwa_klubu, sum(punkty_globalne) as laczna_ilosc_punktow
from bd3_zawodnicy z
            join bd3_wyniki w on z.nr_zawodnika = w.nr_zawodnika
            join bd3_kluby k on z.nr_klubu = k.nr_klubu
            group by imie||' '||nazwisko, plec, data_urodzenia, nazwa_klubu)
where laczna_ilosc_punktow in (select max(laczna_ilosc_punktow) from (select nr_zawodnika, sum(punkty_globalne) as laczna_ilosc_punktow from bd3_wyniki group by nr_zawodnika))
;

---------------------------------------

select Nr_zawodow, nazwa_zawodow, podtytul, data_zawodow, liczba_zawodnikow, sumapkt
from
(select zw.nr_zawodow as Nr_zawodow, nazwa_zawodow, podtytul, data_zawodow, count(w.nr_zawodnika) as liczba_zawodnikow, sum(punkty_globalne) as sumapkt
from bd3_zawody zw
        join bd3_wyniki w on zw.nr_zawodow = w.nr_zawodow 
        join bd3_zawodnicy z on w.nr_zawodnika = z.nr_zawodnika
where (extract(year from sysdate) - extract(year from data_zawodow)) < 12
group by zw.nr_zawodow, nazwa_zawodow, podtytul, data_zawodow
having sum(punkty_globalne) > 2400 and count(w.nr_zawodnika) > 180)
order by liczba_zawodnikow, sumapkt desc
;
  
--------------------------------------------

select imie_nazwisko, liczba_punktow, wiek
from
(select imie||' '||nazwisko as imie_nazwisko, sum(punkty_globalne) as liczba_punktow, round((sysdate-data_urodzenia)/365) as wiek
from bd3_zawodnicy z
            join bd3_wyniki w on z.nr_zawodnika = w.nr_zawodnika
group by imie||' '||nazwisko, round((sysdate-data_urodzenia)/365)
having round((sysdate-data_urodzenia)/365) < 40
order by liczba_punktow desc)
where liczba_punktow is not null and rownum < 6
;

-----------------------------------------------

select imie_nazwisko, sr_liczba_pkt
from
(select imie||' '||nazwisko as imie_nazwisko, round(avg(punkty_globalne)) as sr_liczba_pkt
from bd3_zawodnicy z
            join bd3_wyniki w on z.nr_zawodnika = w.nr_zawodnika 
group by imie||' '||nazwisko
having round(avg(punkty_globalne)) > (select avg(sr) from (select nr_zawodow, avg(punkty_globalne) as sr
                                                        from bd3_wyniki
                                                        group by nr_zawodow))
order by sr_liczba_pkt desc)
where sr_liczba_pkt is not null;

----------------------------------------------

select imie||' '||nazwisko as Imie_nazwisko, rezultat_min||':'||rezultat_sek as wynik, nazwa_klubu, nazwa_kategorii
from bd3_zawodnicy z
            join bd3_wyniki w on z.nr_zawodnika = w.nr_zawodnika
            join bd3_kluby k on z.nr_klubu = k.nr_klubu
            join bd3_kategorie kt on z.nr_kategorii = kt.nr_kategorii
where (plec = 'M') and (nr_zawodow in (1,3)) and ((rezultat_min between 44 and 47) or (rezultat_min = 48 and rezultat_sek = 0))
order by nr_zawodow, wynik;

--------------------------------------------------

select imie||' '||nazwisko as imie_nazwisko, k.nr_klubu, nazwa_klubu, nazwa_kategorii, extract(year from data_urodzenia) as rok_urodzenia
from bd3_zawodnicy z
            join bd3_kluby k on z.nr_klubu = k.nr_klubu
            join bd3_kategorie kt on z.nr_kategorii = kt.nr_kategorii
            join bd3_wyniki w on z.nr_zawodnika = w.nr_zawodnika
where (plec = 'K') and (extract(year from data_urodzenia) between 1975 and 1984) and (punkty_globalne is not null)
;

---------------------------------------------------

select imie||' '||nazwisko as imie_nazwisko, rezultat_min||':'||rezultat_sek, nazwa_klubu, nazwa_kategorii
from bd3_zawodnicy z
            join bd3_wyniki w on z.nr_zawodnika = w.nr_zawodnika
            join bd3_kluby k on z.nr_klubu = k.nr_klubu
            join bd3_kategorie kt on z.nr_kategorii = kt.nr_kategorii
where (plec = 'M') and (nr_zawodow = 3)
order by rezultat_min, rezultat_sek desc
;

-----------------------------------------------------

select imie||' '||nazwisko as imie_nazwisko, plec, extract(year from data_urodzenia) as rok_uro, nazwa_klubu, nazwa_kategorii
from bd3_zawodnicy z
            join bd3_kategorie kt on z.nr_kategorii = kt.nr_kategorii
            join bd3_kluby k on z.nr_klubu = k.nr_klubu
where ((plec = 'M') and (nazwa_kategorii = 'IV')) or ((plec = 'K') and (extract(year from data_urodzenia) between 1970 and 1985) and (nazwisko like 'K%ska'))
order by rok_uro, nazwisko
;

-------------------------------------------------------

select imie||' '||nazwisko as imie_naz, data_wypozyczenia, data_ostatniej_naprawy, s.nr_egzemplarza
from klient k
        join faktura f on k.pesel = f.pesel
        join faktura_dane df on f.nr_faktury = df.nr_faktury
        join wypozyczenia dw on (df.nr_pozycji = dw.nr_pozycji and df.nr_faktury = dw.nr_faktury)
        join sprzet s on dw.nr_egzemplarza = s.nr_egzemplarza
where data_wypozyczenia in (select max(data_wypozyczenia) from (select z.nr_egzemplarza, w.data_wypozyczenia, min(data_ostatniej_naprawy - w.data_wypozyczenia)
                                    from sprzet z
                                            join wypozyczenia w on z.nr_egzemplarza = w.nr_egzemplarza
                                    where z.nr_egzemplarza = s.nr_egzemplarza
                                    group by z.nr_egzemplarza, w.data_wypozyczenia
                                                        having min(data_ostatniej_naprawy - w.data_wypozyczenia) > 0))
and data_ostatniej_naprawy is not null 
group by s.nr_egzemplarza, imie||' '||nazwisko, data_wypozyczenia, data_ostatniej_naprawy
having min(data_ostatniej_naprawy - data_wypozyczenia) > 0
;

------------------------------------------------------

select nazwa_sprzetu, cena, laczny_przychod
from
(select nazwa_sprzetu, cena, sum(cena) as laczny_przychod
from sprzet s
join wypozyczenia w on w.nr_egzemplarza = s.nr_egzemplarza
group by nazwa_sprzetu, cena)
where cena > laczny_przychod * 0.1
;

------------------------------------------------------

select nazwa_kategorii, count(z.nr_zawodnika) as liczba_ucz
from bd3_zawodnicy z 
            join bd3_kategorie kt on z.nr_kategorii = kt.nr_kategorii
            join bd3_wyniki w on z.nr_zawodnika = w.nr_zawodnika
            join bd3_zawody zw on w.nr_zawodow = zw.nr_zawodow
where zw.nr_zawodow = 3
group by nazwa_kategorii
order by liczba_ucz desc
;

----------------------------------------------------

select substr(imie,1,1)||'.'||nazwisko as imie_naz, round((sysdate-data_urodzenia)/365) as wiek, nazwa_kategorii
from bd3_zawodnicy z 
            join bd3_kategorie kt on z.nr_kategorii = kt.nr_kategorii
where (nr_zawodnika between 30 and 60) or (nr_zawodnika between 300 and 600)
order by wiek desc
;

---------------------------------------------------

select count(z.nr_zawodnika) as lk
from bd3_zawodnicy z 
            join bd3_wyniki w on z.nr_zawodnika = w.nr_zawodnika
            join bd3_zawody zw on w.nr_zawodow = zw.nr_zawodow
where (plec = 'K') and (zw.nr_zawodow = 4) and (z.nr_zawodnika between 10 and 20)
;

-------------------------------------------------

select nazwa_klubu, sum(punkty_globalne) as liczba
from bd3_zawodnicy z 
            join bd3_wyniki w on z.nr_zawodnika = w.nr_zawodnika
            join bd3_kluby k on z.nr_klubu = k.nr_klubu
where (nr_zawodow = 1) and (plec = 'M')
group by nazwa_klubu, k.nr_klubu 
having sum(punkty_globalne) is not null
order by liczba desc, k.nr_klubu asc
;

------------------------------------------------

select plec, data_zawodow, nazwa_zawodow, round(avg((sysdate - data_urodzenia)/365), 1) as sr_wiek, count(z.nr_zawodnika) as liczba_uczestnikow
from bd3_zawodnicy z 
            join bd3_wyniki w on z.nr_zawodnika = w.nr_zawodnika
            join bd3_zawody zw on w.nr_zawodow = zw.nr_zawodow
group by plec, data_zawodow, nazwa_zawodow
order by data_zawodow, plec, liczba_uczestnikow
;

------------------------------------------------

select imie||' '||nazwisko as imie_naz, rezultat_min||':'||rezultat_sek as wynik, nazwa_klubu
from bd3_zawodnicy z
            join bd3_wyniki w on z.nr_zawodnika = w.nr_zawodnika
            join bd3_kluby k on z.nr_klubu = k.nr_klubu
where k.nr_klubu in (select nr_klubu from (select nr_klubu, pozycja_globalnie
                                            from bd3_wyniki w 
                                                    join bd3_zawodnicy z on w.nr_zawodnika = z.nr_zawodnika
                                            where (plec = 'M') and (pozycja_globalnie = 1) and (nr_zawodow = 2)))
;

--------------------------------------------------

select imie||' '||nazwisko as imie_naz, nazwa_klubu
from bd3_zawodnicy z
            join bd3_kluby k on z.nr_klubu = k.nr_klubu
where plec = 'K' and nazwa_klubu in (select nazwa_klubu from (select nazwa_klubu, count(nr_zawodnika) as liczba_zaw
                                from bd3_zawodnicy z
                                        join bd3_kluby k on z.nr_klubu = k.nr_klubu
                                group by nazwa_klubu
                                having count(nr_zawodnika) > 5))
;

---------------------------------------------------

select imie_naz, nazwa_klubu, nazwa_kategorii, wiek
from
(select imie||' '||nazwisko as imie_naz, nazwa_klubu, nazwa_kategorii, round((sysdate - data_urodzenia)/365) as wiek
from bd3_zawodnicy z
            join bd3_kategorie kt on z.nr_kategorii = kt.nr_kategorii
            join bd3_kluby k on k.nr_klubu = z.nr_klubu
where (plec = 'K') and (nazwa_klubu not like '%Wars%'))
where wiek  > (select avg(wiek) from (select imie||' '||nazwisko as imie_naz, nazwa_klubu, nazwa_kategorii, round((sysdate - data_urodzenia)/365) as wiek
                                        from bd3_zawodnicy z
                                                    join bd3_kategorie kt on z.nr_kategorii = kt.nr_kategorii
                                                    join bd3_kluby k on k.nr_klubu = z.nr_klubu
                                        where (plec = 'K') and (nazwa_klubu not like '%Wars%')))
;

-----------------------------------------------------

select imie_naz, nazwa_klubu, nazwa_kategorii, li
from
(select imie||' '||nazwisko as imie_naz, nazwa_klubu, nazwa_kategorii, sum(punkty_globalne) as li
from bd3_zawodnicy z
            join bd3_wyniki w on z.nr_zawodnika = w.nr_zawodnika
            join bd3_kategorie kt on z.nr_kategorii = kt.nr_kategorii 
            join bd3_kluby k on z.nr_klubu = k.nr_klubu
group by imie||' '||nazwisko, nazwa_klubu, nazwa_kategorii) 
where li in (select lli from (select plec, max(li) as lli from (select plec, z.nr_zawodnika, sum(punkty_globalne) as li 
                                                                from bd3_wyniki w 
                                                                            join bd3_zawodnicy z on w.nr_zawodnika = z.nr_zawodnika 
                                                                group by z.nr_zawodnika, plec) 
                              group by plec))
;

-----------------------------------------------------