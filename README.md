# SQL-project-Indian-Census-2011-

select * from [indian censes].dbo.Data1;

select * from [indian censes].dbo.[Data 2];

--number of rows into dataset

select count(*) from [indian censes]..Data1
select count(*) from [indian censes]..[Data 2]

--dataset for Andhrapradesh and Maharastra

select * from [indian censes]..Data1 where state in ('Andhra Pradesh','Maharashtra')

--population of India

select * from [indian censes]..[Data 2]
select sum(population) as Population from [indian censes]..[Data 2]

--avg growth percentage

select state,avg(growth)*100 avg_growth from [indian censes]..Data1 group by state;

--avg sex ratio

select state,round(avg(sex_ratio),0) avg_sexratio from [indian censes]..Data1 group by state order by avg_sexratio desc;

--avg litericy rate

select state,round(avg(literacy),0) avg_literacy from [indian censes]..Data1 
group by state having round(avg(literacy),0)>90 order by avg_literacy desc ;

--top 3 states showing highest growth ratio

select top 3 state,avg(growth)*100 avg_growth from [indian censes]..Data1 group by state order by avg_growth desc;

--bottom 3 states showing lowest growth ratio

select top 3 state,avg(growth)*100 avg_growth from [indian censes]..Data1 group by state order by avg_growth asc;

--bottom 3 states showing lowest sex ratio

select top 3 state,round(avg(sex_ratio),0) avg_sexratio from [indian censes]..Data1 group by state order by avg_sexratio asc;

--top and botom 3 states in literacy rate

drop table if exists #topstates;

create table #topstates
( state nvarchar(255),
  topstate float

  )

  insert into #topstates
  select state,round(avg(literacy),0) avg_literacy_ratio from [indian censes]..Data1 
  group by state order by avg_literacy_ratio desc;

  select top 3* from #topstates order by #topstates.topstate desc; 

 drop table if exists #bottomstates;

create table #bottomstates
( state nvarchar(255),
  bottomstate float

  )

  insert into #bottomstates
  select state,round(avg(literacy),0) avg_literacy_ratio from [indian censes]..Data1 
  group by state order by avg_literacy_ratio asc;

  select top 3* from #bottomstates order by #bottomstates.bottomstate asc;

  --union operator

  select * from (
  select top 3* from #topstates order by #topstates.topstate desc)a
  
  union

  select * from(
 select top 3* from #bottomstates order by #bottomstates.bottomstate asc)b; 

 --states starting with lettter a

 select distinct state from [indian censes]..Data1 where lower (state) like 'a%' or lower(state) like 'b%'


 --joining both tables

 select d.state,sum(d.males) total_males,sum(d.females)total_females from
( select c.district,c.state,round(c.population/(c.sex_ratio+1),0)males,round((c.population*c.sex_ratio)/(c.sex_ratio+1),0)females ,c.Population from
( select a.district,a.state,a.sex_ratio/1000 sex_ratio,b.population from [indian censes]..Data1 a inner join [indian censes]..[Data 2] b on a.district=b.district) c)d
group by d.state;


--total literacy rate

select c.state, sum(literate_people) total_literate_people,sum(illeterate_people) total_illeterate_people from
(select d.district,d.state,round(d.literacy_ratio*d.population,0) literate_people ,round((1-d.literacy_ratio)*d.population,0) illeterate_people from
(select a.district,a.state,a.literacy/100 literacy_ratio,b.population from [indian censes]..Data1 a inner join [indian censes]..[Data 2] b on a.district=b.district)d)c
group by c.state;


--population in previous census

select sum(m.state_prev_pop) prev_total_pop,sum(m.state_curr_pop) curr_total_pop from
(select e.state,sum(e.previous_census_population) state_prev_pop , sum(e.current_census_population) state_curr_pop from
(select d.district,d.state,round(d.population/(1+growth),0) previous_census_population,d.population current_census_population from
(select a.district,a.state,a.growth growth,b.population from [indian censes]..Data1 a inner join [indian censes]..[Data 2] b on a.district=b.district)d)e
group by e.state)m


--population vs area

select g.total_area/g.prev_total_pop prev_pop_per_area , g.total_area/g.curr_total_pop cur_pop_per_area from
(select q.*,r. total_area from

(select '1' as keyy,n.* from
(select sum(m.state_prev_pop) prev_total_pop,sum(m.state_curr_pop) curr_total_pop from
(select e.state,sum(e.previous_census_population) state_prev_pop , sum(e.current_census_population) state_curr_pop from
(select d.district,d.state,round(d.population/(1+growth),0) previous_census_population,d.population current_census_population from
(select a.district,a.state,a.growth growth,b.population from [indian censes]..Data1 a inner join [indian censes]..[Data 2] b on a.district=b.district)d)e
group by e.state)m)n) q inner join (

select '1' as keyy,z.*from
(select sum(area_km2) total_area from [indian censes]..[Data 2] )z)r on q.keyy=r.keyy)g


--window

output top 3 districts from each state with highest literacy rate

select a.* from 
(select district,state,literacy,rank() over(partition by state order by literacy desc) rnk from [indian censes]..Data1)a
where a.rnk in (1,2,3) order by state

select a.* from 
(select district,state,literacy,rank() over(partition by state order by literacy asc) rnk from [indian censes]..Data1)a
where a.rnk in (1,2,3) order by state
