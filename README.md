## Inleiding tot de usecase

Een agent wordt getraind om obstakels te vermijden en om rewards op te maken door middel van te springen.

## Observaties, acties en beloning

In het geval van onze agent zijn de observaties diegene die hij maakt door te springen en te zien of hij beloningen krijgt of strafpunten, hij kan ook rewards of obstakels zien aankomen.

## Acties

Wij hebben in dit geval maar 1 actie die de agent kan uitvoeren om zoveel mogelijk punten te halen
  
  - 1 jump actie

## Beloning

Het beloningsmechanisme vertelt het leeralgoritme of de voorgestelde actie de agent dichter bij het einddoel van de leeroefening brengt of niet.

We geven onze agent een beloning van +0.75 als deze een reward pakt en een straf van -1 als deze een obstakel aanraakt. Ook als deze geen reward pakt dan zal deze een reward krijgen van +1 en als de agent een reward mist zal deze een straf krijgen van -0.75.

Verder krijgt onze agent een afstraffing van -0.1 als deze springt om te vermijden dat deze heel de tijd springt.

# Unity omgeving aanmaken

Als eerste zal je een empty object moeten aanmaken genaamd "Environment" waarin een "Plane" object, een "Obstacles" empty, een "Wall" kubus object, een "Agent" kubus object, een "TextMeshPro" object en een "Rewards" empty gestoken moet worden. Dan zal je aan het "Environment" object een C# script moeten koppelen. Hiervoor moet je ook een Obstacle en een Reward Prefab maken met respectievelijk een tag "Obstacle" en een tag "Reward". Deze zullen beide ook een RigidBody en een box Collider moeten hebben en deze zullen dan ook een script moeten hebben.

ObstacleMove.cs op een "obstacle" prefab

```
using System.Collections;
using System.Collections.Generic;
using Unity.MLAgents;
using UnityEngine;

public class ObstacleMove : MonoBehaviour
{

    [SerializeField]
    private float speed = 4f;

    

    private JumperAgent agent;
    // Start is called before the first frame update
    void Start()
    {
        agent = GameObject.Find("Agent").GetComponent<JumperAgent>();
    }

    private void OnEnable()
    {
        //speed = Random.Range(4, 8);
    }

    // Update is called once per frame
    void Update()
    {
        
        transform.localPosition += new Vector3(speed * Time.deltaTime, 0, 0);
    }

    private void OnCollisionEnter(Collision collision)
    {      
            if (collision.transform.CompareTag("Wall"))
            {
               // Debug.Log("raak");
                agent.AddReward(1f);
                Destroy(this.gameObject);
               // agent.EndEpisode();
        }
            
       
    }
}
```
RewardMove.cs op een "Reward Prefab

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class RewardMove : MonoBehaviour
{
    [SerializeField]
    private float speed = 4f;



    private JumperAgent agent;
    // Start is called before the first frame update
    void Start()
    {
        agent = GameObject.Find("Agent").GetComponent<JumperAgent>();
    }

    private void OnEnable()
    {
        //speed = Random.Range(4, 8);
    }

    // Update is called once per frame
    void Update()
    {

        transform.localPosition += new Vector3(speed * Time.deltaTime, 0, 0);
    }

    private void OnCollisionEnter(Collision collision)
    {
        if (collision.transform.CompareTag("Wall"))
        {
            // Debug.Log("raak");
            agent.AddReward(-0.5f);
            Destroy(this.gameObject);
            // agent.EndEpisode();
        }


    }
}
```
Deze zorgen er gewoon voor dat de Reward en het Obstakel blijven bewegen en punten aftrekken of vrijgeven als deze colliden met de wall, deze zullen dan ook verdwijnen.

 Environment jumper.cs

 ``` 
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnvironmentJumper : MonoBehaviour
{
    public GameObject ObstaclePrefab;
    public GameObject Obstacles;
    public GameObject RewardPrefab;
    public GameObject Rewards;
    public bool canSpawnObstacles = true;
    public bool canSpawnRewards = true;

    // Start is called before the first frame update
    void Start()
    {
        
    }

    private void OnEnable()
    {
        Obstacles = transform.Find("Obstacles").gameObject;
        Rewards = transform.Find("Rewards").gameObject;
        Debug.Log("start spwn");
        
        StartCoroutine(SpawnObstacleContinuously());
        StartCoroutine(SpawnRewardsConinuously());

    }
    // Update is called once per frame
    void Update()
    {
        
    }

    private IEnumerator SpawnObstacleContinuously()
    {
        while (true)
        {
            float r = Random.Range(2f, 5.0f);
            yield return new WaitForSeconds(r); 
            if(canSpawnObstacles)
               SpawnObstacle();
        }
    }

    private IEnumerator SpawnRewardsConinuously()
    {
        while (true)
        {
            float r = Random.Range(4f, 8.0f);
            yield return new WaitForSeconds(r);
            if (canSpawnRewards)
                SpawnReward();
        }
    }

    //Spawn every X seconds

    public void SpawnObstacle()
    {
        GameObject newObstacle = Instantiate(ObstaclePrefab.gameObject);

        newObstacle.transform.SetParent(Obstacles.transform);
        // float rx = Random.Range(-4f, 4);
        // float rz = Random.Range(-4f, 2);
        newObstacle.transform.localPosition = new Vector3(-8, 0.5f, 0);
    }

    public void SpawnReward()
    {
        GameObject newReward = Instantiate(RewardPrefab.gameObject);

        newReward.transform.SetParent(Rewards.transform);
        // float rx = Random.Range(-4f, 4);
        // float rz = Random.Range(-4f, 2);
        newReward.transform.localPosition = new Vector3(-8, 3.0f, 0);
    }

    public void ClearEnvironment()
    {        
        foreach (Transform obstacle in Obstacles.transform)
        {
            GameObject.Destroy(obstacle.gameObject);
        }
        foreach (Transform reward in Rewards.transform)
        {
            GameObject.Destroy(reward.gameObject);
        }

        canSpawnObstacles = true;
        canSpawnRewards = true;
    }
}
```
Bij deze worden Rewards en obstakels gecreëerd op willekeurige intervals. En als deze colliden met de agent zal er oftewel een punt afgetrokken worden oftewel een punt vrijgegeven worden.
Je zal dan beide een Obstakel prefab en een reward prefab moeten aanduiden en de Booleans "Can spawn obstacle" en "Can spawn Rewards" op true moeten zetten.

Aan het Agent object zal je verschillende componenten moeten koppelen, onder andere een "Ray Perception sensor 3D", "Behaviour parameters" met Discrete Branch op 1 en Branch size op 2, een "Decision Requester, een "Rigidbody" en een "Box collider". Ook hieraan zal een script gekoppeld moeten worden waar je het TextMeshPro object moet definiëren.

JumperAgent.cs

```
using System.Collections;
using System.Collections.Generic;
using TMPro;
using Unity.MLAgents;
using Unity.MLAgents.Actuators;
using Unity.MLAgents.Sensors;
using UnityEngine;

public class JumperAgent : Agent
{
    [SerializeField]
    private float jumpStrength = 5f;
    [SerializeField]
    private TextMeshPro scoreboard;

    private bool canJump = true;
    private Rigidbody rigidbody;
    private EnvironmentJumper environment;

    // Start is called before the first frame update
    void Start()
    {
        rigidbody = GetComponent<Rigidbody>();
        environment = GetComponentInParent<EnvironmentJumper>();
    }

    // Update is called once per frame
    void Update()
    {
        scoreboard.text = GetCumulativeReward().ToString("f4");
    }

    public override void OnEpisodeBegin()
    {
        environment.ClearEnvironment();
        transform.localPosition = new Vector3(7, 0.5f, 0);
        transform.localRotation = Quaternion.Euler(0, -90, 0f);
    }
    

    public override void OnActionReceived(ActionBuffers actions)
    {
        var vectorAction = actions.DiscreteActions;
        if (vectorAction[0] == 1)
        {
            //punish with small negative award to prevent jumping all the time
            AddReward(-0.01f);
            Jump();
        }
    }

    public override void Heuristic(in ActionBuffers actionsOut)
    {
        //map actions to movement
        var jump = 0;
        if (Input.GetKey(KeyCode.Space))
        {
            jump = 1;
        }
        var discreteActionsOut = actionsOut.DiscreteActions;
        discreteActionsOut[0] = jump;
    }



    private void Jump()
    {
        if (canJump)
        {
            rigidbody.AddForce(new Vector3(0, jumpStrength, 0), ForceMode.VelocityChange);
            canJump = false;
        }
    }

    private void OnCollisionEnter(Collision collision)
    {
        if (collision.transform.CompareTag("Plane"))
        {
            canJump = true;
        }

        if (collision.transform.CompareTag("Obstacle"))
        {
            Debug.Log("collide with obstacle");
            Destroy(collision.gameObject);
            AddReward(-1f);           
        }

        if (collision.transform.CompareTag("Reward"))
        {
            Debug.Log("collide with reward");
            Destroy(collision.gameObject);
            AddReward(0.75f);
        }

    }
}
```
Hierin wordt het duidelijk dat de agent kan springen als actie en als deze met bepaalde objecten colliden zullen er punten afgetrokken of vrijgegeven worden. Ook hier wordt het duidelijk dat de agent enkel kan springen als deze op de grond staat.