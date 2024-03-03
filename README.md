import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;
import javax.persistence.TypedQuery;
import java.util.List;

public class Main {

    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("myPersistenceUnit");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        try {
            tx.begin();

            Department department1 = new Department("IT");
            Department department2 = new Department("HR");
            em.persist(department1);
            em.persist(department2);

            Employee employee1 = new Employee("John", 1000, department1);
            Employee employee2 = new Employee("Alice", 1200, department1);
            Employee employee3 = new Employee("Bob", 1500, department2);
            em.persist(employee1);
            em.persist(employee2);
            em.persist(employee3);

            TypedQuery<Object[]> query = em.createQuery(
                    "SELECT d.name, COUNT(e) FROM Department d LEFT JOIN d.employees e GROUP BY d", Object[].class);
            List<Object[]> results = query.getResultList();
            for (Object[] result : results) {
                String departmentName = (String) result[0];
                long employeeCount = (long) result[1];
                System.out.println("Department: " + departmentName + ", Employee Count: " + employeeCount);
            }

            tx.commit();
        } catch (Exception e) {
            if (tx.isActive()) {
                tx.rollback();
            }
            e.printStackTrace();
        } finally {
            em.close();
            emf.close();
        }
    }
}
