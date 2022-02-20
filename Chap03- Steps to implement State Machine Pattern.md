## Steps to implement State Machine Pattern?

There are a few ways you can implement a state pattern in Unity. The better approach is to have each enemy state by its class, which means with a base class that they inherit from.

But first, you have to create some enums of the enemy states, like this -
```C#
public enum EnemyStateEnum
    {
        None,
        Patrolling,
        Chasing,
        Attacking
    }
```
Now we’ll create an EnemyState class like this -
```C#
public class EnemyStates: MonoBehaviour
    {
        public EnemyView enemyView;

        public virtual void OnStateEnter()
        {
            this.enabled = true;
        }
        public virtual void OnStateExit()
        {
            this.enabled = false;
        }


        public void ChangeState(EnemyStates newState)
        {
            if (enemyView.currentState != null)
                enemyView.currentState.OnStateExit();

            enemyView.currentState = newState;
            enemyView.currentState.OnStateEnter();
        }
    }
```
As we have created our base class which is the EnemyState class. Now we’ll create different states for our enemy tank like - patrolling, chasing, attacking, etc. Keep in mind that while creating states for the enemy, that class must inherit from the base state class.

Like this-
```C#
public class EnemyPatrollingState : EnemyStates
    {
        public override void OnStateEnter()
        {
            base.OnStateEnter();
            enemyView.activeState = EnemyStateEnum.Patrolling;
        }
        private void Update()
        {
            enemyView.controller.Patrol();
        }

        public override void OnStateExit()
        {
            base.OnStateExit();
        }
    }
```
Now we’ll do the same for EnemyChasingState also, like this -
```C#
public class EnemyChasingState : EnemyStates
    {
        private bool canChase;
        public override void OnStateEnter()
        {
            base.OnStateEnter();
            enemyView.activeState = EnemyStateEnum.Chasing;
            Chase();
        }

        public override void OnStateExit()
        {
            base.OnStateExit();
        }
        private void OnTriggerEnter(Collider other)
        {
            if (other.gameObject.GetComponent<TankView>() != null)
            {
                enemyView.SetTankView(other.gameObject.GetComponent<TankView>());
                ChangeState(this);
            }
        }
        private void OnTriggerStay(Collider other)
        {
            if (enemyView.activeState == EnemyStateEnum.Attacking || !canChase) return;


            if (other.gameObject.GetComponent<TankView>() != null)
                Chase();

        }
        private void OnTriggerExit(Collider other)
        {
            if (other.gameObject.GetComponent<TankView>() != null)
            {

                ChangeState(enemyView.patrollingState);
            }
        }
        async private void Chase()
        {
            canChase = false;

            enemyView.navMeshAgent.isStopped = true;
            enemyView.navMeshAgent.ResetPath();
            enemyView.navMeshAgent.SetDestination(enemyView.GetTankTransform().position);
            await new WaitForSeconds(2f);

            canChase = true;
        }
    }
```
Now for EnemyAttackingState - 
```C#
public class EnemyAttackingState : EnemyStates
    {
        public override void OnStateEnter()
        {
            base.OnStateEnter();
            enemyView.activeState = EnemyStateEnum.Attacking;
        }
        public override void OnStateExit()
        {
            base.OnStateExit();
        }
        private void OnTriggerEnter(Collider other)
        {
            if (other.gameObject.GetComponent<TankView>() != null)
            {
                enemyView.navMeshAgent.isStopped = true;
                enemyView.navMeshAgent.ResetPath();
                ChangeState(this);
            }
        }
        private void OnTriggerStay(Collider other)
        {
            if (other.gameObject.GetComponent<TankView>() != null)
            {
                Vector3 lookDir = other.transform.position - enemyView.transform.position;
                if (lookDir != new Vector3(0, 0, 0))
                    RotateTowardsTarget();

                enemyView.controller.Attack();
            }
        }

        private void RotateTowardsTarget()
        {
            enemyView.transform.LookAt(enemyView.GetTankTransform());
        }
        private void OnTriggerExit(Collider other)
        {
            if (other.gameObject.GetComponent<TankView>() != null)
            {
                ChangeState(enemyView.chasingState);
            }
        }
    }
```
As we have all the necessary states of the enemy. Now let’s see how we can implement it in the EnemyView script.

like this -
```C#
public class EnemyView: MonoBehaviour
{
	public EnemyPatrollingState patrollingState;
	public EnemyChasingState chasingState;
  public EnemyAttackingState attackingState;
  [SerializeField] private EnemyStateEnum initialState;
  public EnemyStateEnum activeState;
  public EnemyStates currentState;

	private void Start()
        {
            InitializeState();
        }

	private void InitializeState()
        {
            switch (initialState)
            {
                case EnemyStateEnum.Attacking:
                    currentState = attackingState;
                    break;
                case EnemyStateEnum.Chasing:
                    currentState = chasingState;
                    break;
                case EnemyStateEnum.Patrolling:
                    currentState = patrollingState;
                    break;
                default:
                    currentState = null;
                    break;
            }
            currentState.OnStateEnter();
        }
}
```
Like this, you can implement the State Machine Pattern for the Enemy Tank.

**Note** - Don’t forget to make all the states null, once the enemy tank is destroyed.

---
## Repository of Battle Tank game
In order to test your skills, Checkout the repository below for,

Problem Statement and Solution for the Enemy AI for Enemy Tanks of Battle tank game.

https://github.com/outscal/Unity3D-Enemy-AI-Project
