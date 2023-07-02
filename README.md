using UnityEngine;
using UnityEngine.InputSystem;

public class MOVE : MonoBehaviour
{
    [SerializeField] private float moveSpeed = 5f;
    [SerializeField] private float sprintSpeed = 10f; // Speed when sprinting
    [SerializeField] private float jumpForce = 5f;
    private Rigidbody rb;
    private Vector2 movementInput;
    private bool isJumping = false;
    private bool isSprinting = false;
    private Transform mainCameraTransform;

    private void Awake()
    {
        rb = GetComponent<Rigidbody>();
        rb.useGravity = false; // Disable gravity on the Rigidbody

        // Get the main camera transform
        mainCameraTransform = Camera.main.transform;
    }

    private void OnEnable()
    {
        // Enable the movement input action for keyboard
        InputSystem.EnableDevice(UnityEngine.InputSystem.Keyboard.current);
    }

    private void OnDisable()
    {
        // Disable the movement input action for keyboard
        InputSystem.DisableDevice(UnityEngine.InputSystem.Keyboard.current);
    }

    private void Start()
    {
        LockCursor();
    }

    private void Update()
    {
        if (Keyboard.current.escapeKey.wasPressedThisFrame)
        {
            UnlockCursor();
        }

        movementInput = new Vector2(
            Keyboard.current.dKey.isPressed ? 1f : Keyboard.current.aKey.isPressed ? -1f : 0f,
            Keyboard.current.wKey.isPressed ? 1f : Keyboard.current.sKey.isPressed ? -1f : 0f
        );

        if (Keyboard.current.spaceKey.wasPressedThisFrame)
        {
            isJumping = true;
        }

        // Sprint input
        if (Keyboard.current.leftShiftKey.isPressed && Keyboard.current.wKey.isPressed)
        {
            isSprinting = true;
        }
        else
        {
            isSprinting = false;
        }
    }

    private void FixedUpdate()
    {
        Vector3 forward = mainCameraTransform.forward;
        Vector3 right = mainCameraTransform.right;

        forward.y = 0f;
        right.y = 0f;
        forward.Normalize();
        right.Normalize();

        Vector3 movement = (forward * movementInput.y + right * movementInput.x).normalized;

        // Apply sprint modifier
        float currentMoveSpeed = isSprinting ? sprintSpeed : moveSpeed;

        // Move the player using the movement vector
        rb.velocity = movement * currentMoveSpeed;

        if (isJumping)
        {
            rb.AddForce(Vector3.up * jumpForce, ForceMode.Impulse);
            isJumping = false;
        }
    }

    private void OnCollisionEnter(Collision collision)
    {
        if (collision.gameObject.CompareTag("Ground"))
        {
            // Prevent passing through objects with the "Ground" tag
            rb.velocity = Vector3.zero;
        }
    }

    private void LockCursor()
    {
        Cursor.lockState = CursorLockMode.Locked;
        Cursor.visible = false;
    }

    private void UnlockCursor()
    {
        Cursor.lockState = CursorLockMode.None;
        Cursor.visible = true;
    }
}
