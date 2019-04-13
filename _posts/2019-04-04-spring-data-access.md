---
title: "스프링 데이터 엑세스"
date: 2019-04-04 11:15:00 -0400
categories: java spring
---

## 
스프링 레시피 9장 데이터 엑세스의 전반부는 JDBC 사용에 대해 설명하고 후반부는 ORM(JPA)에 대해 설명한다.

전반부인 JDBC의 내용은 JDBC API를 직접 사용하는 방법이 반복되는 코드의 호출이 많아 불편하고 오류를 발생시킬 수 있다는 점을 알려주고, JDBC API를 보다 손쉽게 사용할 수 있도록 스프링에서 제공하는 JDBC 템플릿을 사용하는 방법에 대해 알려준다.

***전반부 내용: JDBC는 DB제작사에 구애 받지 않고 DBMS에 엑세스 할 수 있는 API이다. JDBC API를 직접 사용하지 말고 JDBC Template을 사용하라.***

***후반부 내용: ORM은 클래스와 테이블을 매핑하는 기술이다.***

## JDBC를 직접 다룰 때의 문제점
JDBC를 직접 사용하면 지루하고 장황한 API 호출이 반복되며 그때마다 큰 의미가 없는 과정이 되풀이 된다.

**JDBC API를 직접 다루어 DB insert, update, delete 시 반복되는 작업**
1. 데이터 소스에서 DB 접속을 얻는다.
2. PreparedStatement 객체를 생성한다.
3. 매개변수를 PreparedStatement 객체에 바인딩 한다.
4. PreparedStatement를 실행한다.
5. SQLException을 처리한다.
6. 구문 및 접속 객체를 삭제한다.

**JDBC API를 직접 다루어 DB select 시 반복되는 작업**
1. 데이터 소스에서 DB 접속을 얻는다.
2. PreparedStatement 객체를 생성한다.
3. 매개변수를 PreparedStatement 객체에 바인딩 한다.
4. PreparedStatement를 실행한다.
5. 반복된 ResultSet을 순회한다.
6. ResultSet에서 데이터를 꺼낸다.
7. SQLException을 처리한다.
8. 구문 및 접속 객체를 삭제한다.

다음은 JDBC를 직접 사용한 DAO 클래스이다. 위에서 나열한 작업들이 반복되고 있다.
```java
public class PlainJdbcVehicleDao implements VehicleDao {

    private static final String INSERT_SQL = "INSERT INTO VEHICLE (COLOR, WHEEL, SEAT, VEHICLE_NO) VALUES (?, ?, ?, ?)";
    private static final String UPDATE_SQL = "UPDATE VEHICLE SET COLOR=?,WHEEL=?,SEAT=? WHERE VEHICLE_NO=?";
    private static final String SELECT_ALL_SQL = "SELECT * FROM VEHICLE";
    private static final String SELECT_ONE_SQL = "SELECT * FROM VEHICLE WHERE VEHICLE_NO = ?";
    private static final String DELETE_SQL = "DELETE FROM VEHICLE WHERE VEHICLE_NO=?";

    private final DataSource dataSource;
    
    public PlainJdbcVehicleDao(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public void insert(Vehicle vehicle) {
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(INSERT_SQL)) {
            prepareStatement(ps, vehicle);
            ps.executeUpdate();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public Vehicle findByVehicleNo(String vehicleNo) {
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(SELECT_ONE_SQL)) {
            ps.setString(1, vehicleNo);

            Vehicle vehicle = null;
            try (ResultSet rs = ps.executeQuery()) {
                if (rs.next()) {
                    vehicle = toVehicle(rs);
                }
            }
            return vehicle;
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public List<Vehicle> findAll() {
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(SELECT_ALL_SQL);
             ResultSet rs = ps.executeQuery()) {

            List<Vehicle> vehicles = new ArrayList<>();
            while (rs.next()) {
                vehicles.add(toVehicle(rs));
            }
            return vehicles;
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    private Vehicle toVehicle(ResultSet rs) throws SQLException {
        return new Vehicle(rs.getString("VEHICLE_NO"),
                rs.getString("COLOR"), rs.getInt("WHEEL"),
                rs.getInt("SEAT"));
    }

    private void prepareStatement(PreparedStatement ps, Vehicle vehicle) throws SQLException {
        ps.setString(1, vehicle.getColor());
        ps.setInt(2, vehicle.getWheel());
        ps.setInt(3, vehicle.getSeat());
        ps.setString(4, vehicle.getVehicleNo());
    }

    @Override
    public void update(Vehicle vehicle) {
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(UPDATE_SQL)) {
            prepareStatement(ps, vehicle);
            ps.executeUpdate();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public void delete(Vehicle vehicle) {
        try (Connection conn = dataSource.getConnection();
             PreparedStatement ps = conn.prepareStatement(DELETE_SQL)) {
            ps.setString(1, vehicle.getVehicleNo());
            ps.executeUpdate();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }
}
```

## JDBC 템플릿으로 DB 수정하기
JDBC API는 너무 저수준(low-level)이기 때문에 JDBC 템플릿을 이용하면 꼭 필요한 작업만 더 효과적으로 간단명료하게 나타내면서 안전하게 수행할 수 있다. 

**JDBC 템플릿으로 DB를 수정할때는 update() 메서드를 사용한다.**

JDBC API를 직접 사용했을 때는, DB Connection을 연결하고 해제하는 작업, PreparedStatement로 쿼리를 준비하고 실행시키는 과정, 예외를 처리하는 과정이 필요했지만 JDBC 템플릿을 사용하면 DB Connection은 JDBC 템플릿 객체의 생성과 종료와 함께 처리되고 예외처리도 JDBC 템플릿의 내부 메소드에서 throws Exception으로 처리해주므로 훨씬 코드가 깔끔해진다. 

JDBC API의 PreparedStatement 객체를 생성하고 매개변수를 PreparedStatement에 바인딩하는 것을 JDBC 템플릿의 update() 메서드에서는 인터페이스 사용하여 PreparedStatement를 생성하거나 매개변수를 바인딩한다.

*PreparedStatemenetCreator 인터페이스를 사용*
```java
@Override
public void insert(Vehicle vehicle) {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(this.dataSource);
    jdbcTemplate.update(new PreparedStatementCreator() {
        @Override
        public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
            PreparedStatement ps = con.prepareStatement(INSERT_SQL);
            ps.setString(1, vehicle.getColor());
            ps.setInt(2, vehicle.getWheel());
            ps.setInt(3, vehicle.getSeat());
            ps.setString(4, vehicle.getVehicleNo());
            return ps;
        }
    });
}
```

*PreparedStatementSetter 인터페이스를 사용*
```java
@Override
public void insert(Vehicle vehicle) {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(this.dataSource);
    jdbcTemplate.update(INSERT_SQL, new PreparedStatementSetter() {
        @Override
        public void setValues(PreparedStatement ps) throws SQLException {
            ps.setString(1, vehicle.getColor());
            ps.setInt(2, vehicle.getWheel());
            ps.setInt(3, vehicle.getSeat());
            ps.setString(4, vehicle.getVehicleNo());
        }
    });
}
```

*매개변수 값으로 사용*
```java
@Override
public void insert(Vehicle vehicle) {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(this.dataSource);
    jdbcTemplate.update(INSERT_SQL, vehicle.getColor(), vehicle.getWheel(), vehicle.getSeat(), vehicle.getVehicleNo());
}
```



***JDBC의 update() 메소드를 반복적으로 사용할때는 for문을 돌리지말고 batchUpdate() 메서드를 사용하라.***
```java
@Override
public void insert(Collection<Vehicle> vehicles) {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(this.dataSource);
    jdbcTemplate.batchUpdate(INSERT_SQL, vehicles, vehicles.size(), this::prepareStatement);
}
```

## JDBC 템플릿으로 DB 조회하기
JDBC API를 이용하는 과정에서 비즈니스 로직과 연관된 곳은 쿼리를 정의하고 ResultSet에서 결과를 꺼내는 부분이다. JDBC 템플릿을 사용하면 비즈니스 로직과 관련된 부분만 처리해 주면 된다.

**Jdbc 템플릿으로 DB를 조회할 때는 query() 메서드를 사용한다.**

### 로우 콜백 핸들러로 데이터 추출하기
query() 메서드가 ResultSet을 반환하면 결과 로우를 하나씩 순회하면서 각각 RowCallbackHandler의 processRow()메서드를 호출하여 필요한 작업을 수행한다.
```java
@Override
public Vehicle findByVehicleNo(String vehicleNo) {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
    final Vehicle vehicle = new Vehicle();
    jdbcTemplate.query(SELECT_ONE_SQL, new RowCallbackHandler() {
        @Override
        public void processRow(ResultSet rs) throws SQLException {
            vehicle.setVehicleNo(rs.getString("VEHICLE_NO"));
            vehicle.setColor(rs.getString("COLOR"));
            vehicle.setWheel(rs.getInt("WHEEL"));
            vehicle.setSeat(rs.getInt("SEAT"));
        }
    }, vehicle);
    return vehicle;
}
```

### 로우 매퍼로 데이터 추출하기

*단일 데이터 추출*
```java
@Override
public Vehicle findByVehicleNo(String vehicleNo) {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
    return jdbcTemplate.queryForObject(SELECT_ONE_SQL, new RowMapper<Vehicle>() {
        @Override
        public Vehicle mapRow(ResultSet rs, int rowNum) throws SQLException {
            return new Vehicle(rs.getString("VEHICLE_NO"),
                    rs.getString("COLOR"),
                    rs.getInt("WHEEL"),
                    rs.getInt("SEAT"));
        }
    }, vehicleNo);
}
```

*다중 데이터 추출*
```java
@Override
public List<Vehicle> findAll() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
    return jdbcTemplate.query(SELECT_ALL_SQL, new RowMapper<Vehicle>() {
        @Override
        public Vehicle mapRow(ResultSet rs, int rowNum) throws SQLException {
            return new Vehicle(rs.getString("VEHICLE_NO"),
                    rs.getString("COLOR"),
                    rs.getInt("WHEEL"),
                    rs.getInt("SEAT"));
        }
    });
}
```

### QueryForList 메서드를 사용하여 여러 로우 쿼리하기
QueryForList() 메서드를 사용하면 RowMapper<T> 없이도 SQL 결과 값을 리스트로 받을 수 있다.

```java
@Override
public List<Vehicle> findAll() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
    List<Map<String, Object>> rows = jdbcTemplate.queryForList(SELECT_ALL_SQL);
    return rows.stream().map(row -> {
        Vehicle vehicle = new Vehicle();
        vehicle.setVehicleNo((String) row.get("VEHICLE_NO"));
        vehicle.setColor((String) row.get("COLOR"));
        vehicle.setWheel((Integer) row.get("WHEEL"));
        vehicle.setSeat((Integer) row.get("SEAT"));
        return vehicle;
    }).collect(Collectors.toList());
}
```

### BeanPropertyRowmapper<T> 클래스를 사용하여 단일/다중 로우 쿼리하기
스프링에는 BeanPropertyRowMapper<T>라는 편리한 RowMapper<T> 하위 클래스가 있어서 로우를 특정 클래스로 자동 매핑할 수 있다.

*단일 매핑*
```java
@Override
public Vehicle findByVehicleNo(String vehicleNo) {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
    return jdbcTemplate.queryForObject(SELECT_ONE_SQL, BeanPropertyRowMapper.newInstance(Vehicle.class), vehicleNo);
}
```

*다중 매핑*
```java
@Override
public List<Vehicle> findAll() {
    JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
    return jdbcTemplate.query(SELECT_ALL_SQL, BeanPropertyRowMapper.newInstance(Vehicle.class));
}
```

### ResultSet이 로우 하나 + 컬럼 하나 인 경우 쿼리하기
queryForObject() 메소드를 사용하고 반환 타입을 쿼리 결과값에 맞는 클래스로 넣어준다.
```java
@Override
public int countAll() {
    return jdbcTemplate.queryForObject(COUNT_ALL_SQL, Integer.class);
}
```

## JDBC 템플릿 주입하기
필요할 때마다 JdbcTemplate 인스턴스를 새로 만들면 생성문을 반복하며 객체를 생성해야 하므로 비용 면에서 비효율 적이다. JdbcTemplate는 스레드-안전한(thread-safe) 클래스여서 IoC 컨테이너에 인스턴스를 하나만 선언하고 모든 DAO 인스턴스에 주입해 쓸 수 있다.

*JDBC 템플릿을 주입받을 DAO 클래스*
```java
public class JdbcVehicleDao implements VehicleDao {
    ...
    private final JdbcTemplate jdbcTemplate;

    public JdbcVehicleDao(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    ...
}
```

*jdbc 템플릿을 빈으로 등록하고 DAO 클래스에 주입*
```java
@Bean
public VehicleDao vehicleDao(JdbcTemplate jdbcTemplate) {
    return new JdbcVehicleDao(jdbcTemplate);
}
@Bean
public JdbcTemplate jdbcTemplate(DataSource dataSource) {
    return new JdbcTemplate(dataSource);
}
```

## JdbcDaoSupport 클래스를 상속받고 setJdbcTemplate() 메소드를 사용하여 JDBC 템플릿 주입하기

스프링 프레임워크에서는 JdbcDaoSupport 라는 추상 클래스를 제공해주는데, 이 추상 클래스의 맴버 변수는 JdbcTemplate 하나이다. 그리고 이 추상 클래스는 JdbcTemplate의 관리를 지원한다.

***데이터베이스와 연결을 맺고 끝는 부분 기능을 구현할 때 JdbcDaoSupport 추상클래스를 상속받아 사용하면 편할 것 같다.***

*스프링에서 제공하는 JdbcDaoSupport 추상 클래스*
```java
public abstract class JdbcDaoSupport extends DaoSupport {

	@Nullable
	private JdbcTemplate jdbcTemplate;

	public final void setDataSource(DataSource dataSource) {
		if (this.jdbcTemplate == null || dataSource != this.jdbcTemplate.getDataSource()) {
			this.jdbcTemplate = createJdbcTemplate(dataSource);
			initTemplateConfig();
		}
	}

    protected JdbcTemplate createJdbcTemplate(DataSource dataSource) {
		return new JdbcTemplate(dataSource);
	}

    @Nullable
	public final DataSource getDataSource() {
		return (this.jdbcTemplate != null ? this.jdbcTemplate.getDataSource() : null);
	}

	public final void setJdbcTemplate(@Nullable JdbcTemplate jdbcTemplate) {
		this.jdbcTemplate = jdbcTemplate;
		initTemplateConfig();
	}
    
    @Nullable
	public final JdbcTemplate getJdbcTemplate() {
	  return this.jdbcTemplate;
	}

    protected void initTemplateConfig() {
	}

    @Override
	protected void checkDaoConfig() {
		if (this.jdbcTemplate == null) {
			throw new IllegalArgumentException("'dataSource' or 'jdbcTemplate' is required");
		}
	}

	protected final SQLExceptionTranslator getExceptionTranslator() {
		JdbcTemplate jdbcTemplate = getJdbcTemplate();
		Assert.state(jdbcTemplate != null, "No JdbcTemplate set");
		return jdbcTemplate.getExceptionTranslator();
	}

	protected final Connection getConnection() throws CannotGetJdbcConnectionException {
		DataSource dataSource = getDataSource();
		Assert.state(dataSource != null, "No DataSource set");
		return DataSourceUtils.getConnection(dataSource);
	}
    
    protected final void releaseConnection(Connection con) {
		DataSourceUtils.releaseConnection(con, getDataSource());
	}
}
```

JdbcDaoSupport를 상속받은 Dao 클래스에서 JdbcTemplate 빈을 주입받을 때는 다음과 같이 Dao 클래스의 생성자에서 JdbcDaoSupport 추상 클래스의 setJdbcTemplate 메소드를 호출하여 JdbcTemplate을 주입해주면 된다.

*JdbcDaoSupport를 상속받은 Dao 클래스에서의 JdbcTemplate 빈 주입*
```java
public class JdbcVehicleDao extends JdbcDaoSupport implements VehicleDao {
    ...
    public JdbcVehicleDao(JdbcTemplate jdbcTemplate) {
        setJdbcTemplate(jdbcTemplate);
    }
    ...
}
```




***
참고도서  
제목: 스프링5레시피(4판)  
지은이: 마틴데니엄, 다니엘 루비오, 조시 롱  
옮긴이: 이일웅  
펴낸곳: 한빛미디어  
